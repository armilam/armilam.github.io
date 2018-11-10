---
layout:             post
title:              Use a return_url with omniauth
date:               2015-03-24 15:55:54 -0400
tags:               
---

I've been working on implementing single sign on in Lessonly (a Rails 4.2 app), specifically with SAML (because that's what the clients need). [Omniauth](https://github.com/PracticallyGreen/omniauth-saml) has been making it fairly easy to do, which has been great.

However, I had need of keeping track of a `return_url`. This can get weird because with SSO, the service provider (SP) sends the user to a different place on the web, the identity provider (IdP), which then sends the user back to your SP. Keeping track of a `return_url` is not reliable through these requests.

There is one thing that I tend not to think about when developing Rails apps - the session. In general, I believe that using the session is a bad idea. The big exception being that I use it to keep track of the current logged in user.

Since I don't generally use the session, this option escaped my mind for a while when I was trying to figure out this particular problem. However, it works very well here.

Omniauth allows us to define a setup action which runs before the authentication request is sent:

```rb
# config/initializers/omniauth.rb
provider :saml, setup: true

# config/routes.rb
get "/auth/:provider/setup" => "sessions#sso_setup"

# controllers/sessions_controller.rb
def sso_setup
  session[:sso_return_url] = params[:return_url] if params[:return_url]
  # additional setup...
end
```

We can then use the return_url in the session upon authentication:

```rb
# config/routes.rb
post "/auth/:provider/callback" => "sessions#sso", as: :sso_callback

# controllers/sessions_controller.rb
def sso
  # authenticate...
  redirect_to sessions[:return_url]
end
```

Just as simple as that.
