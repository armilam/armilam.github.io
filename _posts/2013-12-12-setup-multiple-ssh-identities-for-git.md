---
layout: post
title:  Setup multiple SSH identities for git
date:   2013-12-02 04:45:00 -0400
tags:   
---

Today, I learned how to set up multiple SSH identities out of necessity. I use the same machine for development at SEP and for development of my own personal projects. I use Bitbucket, Github, and others for my code repositories as well as SEP's own Gitorious server for internal and client projects.

I'd rather use different SSH identities for each of the SCM servers I use, so today I found out just how to do that - at least on Mac OS X. I believe these instructions will be very similar for Linux and Unix.

First, generate your SSH keys as per your usual method in your .ssh directory (/Users/armilam/.ssh for me). Name each one with a unique and easy to recognize name. I have id_rsa_github, id_rsa_bitbucket, and others.

Then, create a 'config' file in your .ssh directory. For each identity, you should have one entry. I have a fairly simple setup. My config file looks like this:

```
Host github-work
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_github_work

Host github-personal
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_github_personal

Host bitbucket
  HostName bitbucket.org
  IdentityFile ~/.ssh/id_rsa_bitbucket
```

And to finish it off, I edit the origin URLs in each of my repositories' config files. I can edit the domain portion of each to the host alias given in the .ssh/config file so that git knows which ssh key to use. For example:

```
url = ssh://git@github.com/armilam/project-name.git
```

becomes

```
url = ssh://git@github-personal/armilam/project-name.git
```

Simple as that. Configuration for Mercurial is similar.
