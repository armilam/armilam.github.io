---
layout: post
title: We Migrated from AGIC to AGC and Rebuilt Production in the Process
date: 2026-04-14 00:00:00 -0400
tags: [engineering, infrastructure]
---

# We Migrated from AGIC to AGC and Rebuilt Production in the Process

For about three months, a big chunk of my engineering attention went into migrating our AKS clusters off AGIC and onto AGC. What started as "swap one ingress controller for another" turned into a network plugin migration, a Workload Identity rollout, an unplanned outage, and ultimately a full production cluster rebuild in a new region. If you're on AGIC, it'll happen to you eventually. Here's what I wish I'd known.

## Why this was happening at all

AGIC - the Application Gateway Ingress Controller - is Azure's older Kubernetes ingress story. It depends on AAD Pod Identity, which reached end-of-support in September 2025. We were already running on unsupported infrastructure when we finally started this work in January 2026.

AGC (Application Gateway for Containers) is Microsoft's replacement. It's a different Azure service entirely, not just a new version of Application Gateway. New CRDs, new controller, new identity model, new everything.

Beyond the end-of-support pressure, we had a real capacity problem. AGC has a hard 200-routing-rule-per-resource limit. Our dev and stage clusters shared one AGC and had quietly accumulated 207 combined rules. We didn't know about the limit, and AGC gave us no warning. It just stopped registering new rules. The symptom was weird: API endpoints started returning UI HTML. The catch-all rule for the frontend still matched, but the more specific API rule had silently failed to register. That kind of invisible failure will drive you sideways for a while before you find the root cause.

So the migration wasn't optional, and we weren't starting from a clean state.

## The first surprise: disabling the AGIC addon deletes the Application Gateway

On day four of the dev/stage migration, we ran `az aks disable-addons --addons ingress-appgw`. We expected it to stop the AGIC controller pod. Instead, Azure tore down everything the addon had created - the Application Gateway itself, its managed identity, the IngressClass. All of it, gone. The Azure Activity Log confirmed a Delete operation on the Application Gateway resource starting at 16:43:23Z.

Complete outage on dev and stage. The recovery required re-enabling the addon, which created a _new_ Application Gateway with a _new_ public IP, and then chasing DNS to point at the new address.

That one experience changed the entire shape of the migration plan. The original approach was to incrementally swap AGIC addon → AGIC Helm → AGC. We threw that out immediately. The new rule: AGC must be fully serving traffic before you touch the addon. Not "mostly working." Fully serving. Because the moment you disable the addon, the old Application Gateway vanishes.

## The second surprise: HealthCheckPolicy is not optional

Once we got AGC ingresses deployed, every request came back `503 - no healthy upstream`. Status said `ValidIngress` and `Accepted`. The TLS cert was valid. Pods were healthy. Everything looked correct, and nothing worked.

The root cause is that AGC's default health probe hits `/` on port 80. Our ASP.NET services respond to `/health/ready`. With no per-service configuration, AGC marks every backend unhealthy - silently, with no particularly obvious indication that health checking is the problem.

AGIC let you handle this with an annotation on the ingress: `appgw.ingress.kubernetes.io/health-probe-path`. AGC doesn't work that way. You need a `HealthCheckPolicy` CRD per service:

```yaml
apiVersion: alb.networking.azure.io/v1
kind: HealthCheckPolicy
metadata:
  name: my-api-healthcheck
spec:
  targetRef:
    group: ""
    kind: Service
    name: my-api
  default:
    healthBackend:
      port: 80
      path: /health/ready
      statusCodes:
        - start: 200
          end: 299
```

For UI services serving redirects, you need to accept 300-399. For services on non-standard ports, you need to specify the port. We ended up with 72 HealthCheckPolicy resources across two namespaces. The diagnostic that finally pointed us at the real problem was creating one policy for a single service and watching that route instantly start working. Everything else clicked into place once we knew what we were looking for.

The practical lesson: codify HealthCheckPolicy resources into your manifest generator on day one. Don't treat them as an afterthought.

## The third surprise: kubenet is a hard blocker, not a tuning issue

AGC routes traffic directly to pod IPs. With kubenet, pod IPs come from a non-VNet-routable overlay. AGC literally can't reach them. This isn't a configuration problem you can work around. It's a fundamental incompatibility.

We had to migrate from kubenet to Azure CNI Overlay before AGC would work properly. I expected this to be one `az aks update` command. It was three sequential blockers:

Our AGIC subnet was a `/16`. CNI Overlay requires AGIC subnets to be `/24` or smaller. There's no way to resize a subnet in place if it has resources in it.

The kubenet-era route table was still associated to the AGIC subnet and had to be manually disassociated.

We were running Calico for network policy. Azure won't let you upgrade kubenet → CNI Overlay with Calico enabled. Disabling Calico reimages every node, which takes around 15 minutes.

Each of these required its own diagnosis. And none of it is reversible. Once you're on CNI Overlay you cannot go back to kubenet without rebuilding the cluster.

There was also a vCPU quota issue. The CNI Overlay upgrade reimages every node, which requires surge capacity. Our subscription quota didn't have room for the default surge, so we had to lower `--max-surge` on every node pool just to make the upgrade fit. And when we passed `--pod-cidr` to specify a custom CIDR, Azure silently ignored it and used the default. No warning.

## What we did for production: a full rebuild

By the time dev and stage were done, we had a clear picture of what the in-place migration actually cost: four irreversible operations (disable Calico, disable AGIC, remove route table associations, upgrade CNI), estimated 20-45 minutes of downtime, and no meaningful rollback path mid-flight.

We rejected that approach for production. Instead, we built a parallel cluster from scratch - new Kubernetes version, Azure CNI Overlay from day one, Workload Identity enabled at creation, a dedicated NAT Gateway with a static egress IP, memory-optimized VMs to match our actual resource profile, and a region change to co-locate with the database. Then we cut over with a single DNS change.

The static egress IP was put in a separate networking resource group specifically so it survives future cluster rebuilds. That kind of thinking - where does this resource belong so it's not accidentally deleted with the cluster - pays dividends in exactly the moments when you're most distracted.

### Validating before cutover

We gave the new cluster a test hostname and ran it at full production scale for 24 days before the DNS cutover. Every service at full replica count, real production configuration, all the same image digests. No pod restarts, no OOMKills.

A week before cutover, we swapped the ingresses to use the production hostname but kept DNS pointing at the old cluster. Then we verified end-to-end with:

```sh
curl --resolve k8s.example.com:443:<new-agc-ip> https://k8s.example.com/api/healthz/version
```

That's the key move. Confirm the new cluster handles production traffic patterns correctly, with the production hostname, before you touch DNS.

We also pre-warmed services with a couple curl passes across every endpoint. Cold .NET JIT plus cold EF Core model compilation plus cold connection pools added 1-2 seconds to first requests. Two passes of curl dropped second-pass response times under 300ms. That's a small thing that matters a lot if real users are the ones paying the cold-start tax.

DNS TTL was lowered more than 24 hours before cutover. The cutover itself was one DNS change, and the old cluster stayed up and untouched for about a week afterward as a safety net.

## Other gotchas worth knowing

**The ALB Controller crashes immediately if Gateway API CRDs aren't pre-installed.** It doesn't surface a useful error. It just enters CrashLoopBackOff with a fatal log about failing to watch `GRPCRoute` resources. Install the upstream Gateway API CRDs before you deploy the controller. Also check for stale CRDs from older Gateway API versions - a `storedVersions` mismatch on a CRD will block reapplication.

**The `alb-frontend` annotation takes a name, not a resource ID.** `alb-id` takes the full Azure resource ID of the AGC. `alb-frontend` takes just the frontend name. If you use a resource ID for `alb-frontend` (which seems reasonable given `alb-id`), the controller generates a malformed URL. If you omit `alb-frontend` entirely, the controller silently creates a new frontend per ingress. Pin both, and get the format right on each.

**cert-manager needs a separate ClusterIssuer for AGC.** The legacy `class: azure/application-gateway` field in your HTTP-01 solver only routes through AGIC. AGC ignores it. You need a new ClusterIssuer that uses `ingressClassName: azure-alb-external` and an `ingressTemplate` block that injects the AGC annotations onto the solver ingress. The `alb-id` annotation is cluster-specific, which means this ClusterIssuer is also cluster-specific - it can't be shared.

**AKS-created resources live in the `MC_` resource group, not the cluster's resource group.** You know this, and then you forget it at the worst possible moment. Write it on a sticky note.

**PIM-elevate before you start.** Assigning the necessary roles to the ALB Controller's managed identity requires `User Access Administrator` on multiple resource groups. Hitting an `AuthorizationFailed` mid-migration while waiting for PIM elevation to process is not where you want to be.

**AGC frontend addresses are FQDNs, not IPs.** CNAME the frontend FQDN - the underlying IP can change because the service is managed. Don't A-record it.

## What actually worked

The parallel-cluster strategy made the production cutover almost boring. Single DNS change, rollback was one DNS edit, old cluster sitting there untouched as a fallback. Compare that to the in-place approach: four sequential irreversible operations, 20-45 minutes of exposure, no realistic rollback once you're halfway through.

24 days of soak time at full scale was probably overkill. It was also exactly right. By the time we cut over, confidence was high. We weren't hoping the new cluster would handle production load - we'd watched it do it for three weeks.

Deploying with pinned image digests meant the new cluster ran byte-identical images to the old one. No drift, no version surprises. When you're validating a new cluster, you want to be confident the delta is the infrastructure, not the application.

The thing I keep coming back to: in-place migrations of clusters with irreversible operations are riskier than they look going in. The risk isn't that you don't know what you're doing. It's that the dependencies chain in ways you can't fully anticipate until you're standing in the middle of them with prod traffic hitting something that used to work. Building a parallel cluster and cutting over with DNS bounds your blast radius to the TTL window and keeps the old environment intact as a real rollback. That's worth the extra setup.

Do it once and you'll never do it the other way again.
