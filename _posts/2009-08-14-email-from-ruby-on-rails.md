---
layout:             post
title:              Email From Ruby on Rails
date:               2009-08-14 13:05:00 -0400
tags:               
---

Who knew that sending an email using ActionMailer and TLS would be so difficult? It turns out that ActionMailer isn't equipped to work with SSL connections to SMTP servers. There is, however, a plugin created to get ActionMailer to work with TLS [here](http://agilewebdevelopment.com/plugins/actionmailer_tls).

The official documentation for ActionMailer can be found [here](http://api.rubyonrails.org/classes/ActionMailer/Base.html) and a good tutorial can be found [here](https://we.riseup.net/rails/actionmailer). Once you add the TLS plugin, the only difference you need to make is to set

```
:tls => true
```

when you set

```
config.action_mailer.smtp_setting = { ... }
```

in environment.rb. And be sure to use the correct port for your email server.
