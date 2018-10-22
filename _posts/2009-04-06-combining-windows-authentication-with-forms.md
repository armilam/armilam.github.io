---
layout:             post
title:              Combining Windows Authentication with Forms Authentication in ASP.NET
date:               2009-04-06 16:13:00 -0400
tags:               
---

The problem I am working on now involves combining Windows Authentication with Forms Authentication in IIS. This took a good deal of trial and error, so I'll post my solution here.

I started with the solutions found here:
- [http://msdn.microsoft.com/en-us/library/ms972958.aspx](http://msdn.microsoft.com/en-us/library/ms972958.aspx)
- [http://aspadvice.com/blogs/rjdudley/archive/2005/03/10/2562.aspx](http://aspadvice.com/blogs/rjdudley/archive/2005/03/10/2562.aspx)

## Overview

It seems most implementations will involve the following setup (as it was in my case):
- There is a website that users must login to view
- Users on the local intranet are signed in to their machines with their Active Directory account, and should be able to login via Windows Authentication
- Users outside the network should be directed to a sign in page

Since Windows and Forms Authentication can't really be combined in IIS, you have to fake it. Have Forms Authentication configured for the site. Detect when a user is connecting from within the local network and route those users to a special page configured to login users programmatically.

## Configure Site for Forms Authentication

Start by configuring your site for Forms Authentication in IIS.
1. Right-click the virtual directory in IIS Manager and click Properties. Click the Edit button in the Directory Security tab. Make sure Anonymous Access is allowed.
2. Setup Web.config for Forms Authentication.
    ```xml
    <authentication mode="Forms">
      <forms loginUrl="Login.aspx" name=".ASPXAUTH" timeout="60" path="/">
      </forms>
    </authentication>
    
    <identity impersonate="true" userName="domainusername" password="password"/>
    
    <authorization>
      <deny users="?"/>
      <allow users="*"/>
    </authorization>
    ```
3. Login.aspx should be your page with username and password fields, with login logic in the Login_click(...) function. I won't go into detail here as this is a fairly common function. As long as a FormsAuthenticationTicket is made and the .ASPXAUTH cookie is set to the encrypted ticket, everything is good.

## Add Windows Authentication

Add Windows Authentication by tricking IIS a little. A new login form WinLogin.aspx will capture the client's Windows identity and perform Forms Authentication using that.
1. Add WinLogin.aspx to the same directory as Login.aspx. It doesn't have any UI code as it just authenticates the user and redirects them. The code I use is below.
    ```csharp
    protected override void OnPreRender(EventArgs e)
    {
      string username = Request.ServerVariables["LOGON_USER"];
      if( username != null && username.Length > 0 )
      {
        GenericIdentity id = new GenericIdentity(userid);

        FormsAuthenticationTicket ticket =
            new FormsAuthenticationTicket(
                1,
                userid,
                DateTime.Now,
                DateTime.Now.AddMinutes(20),
                false,
                Guid.NewGuid().ToString()); 

        // Encrypt the ticket
        string encrypted_ticket = FormsAuthentication.Encrypt(ticket);

        // Create cookie
        HttpCookie cookie = new HttpCookie(
            FormsAuthentication.FormsCookieName,
            encrypted_ticket); 

        // Add cookie
        HttpContext.Current.Response.Cookies.Add(cookie);
      }
      else
      {
        Response.Redirect( "Login.aspx" );
      }
    }
    ```
2. Right-click WinLogin.aspx in IIS Manager and click Properties. Click the Edit button in the Directory Security tab. Disallow anonymous access for this file and require Integrated Windows authentication. Doing this will allow WinLogin.aspx to capture the Windows user's identity.
3. Add a location for WinLogin.aspx to Web.config to allow unauthenticated users to access it.
    ```xml
    <location path="WinLogin.aspx">
      <system.web>
        <authorization>
          <allow users="*"/>
        </authorization>
      </system.web>
      </location>
    ```
4. Add a redirect in Login.aspx. This part I got from [Richard Dudley](http://msdn.microsoft.com/en-us/library/ms972958.aspx). This really works well because a user will never have to see the ugly web browser Windows Authentication alert.

Simply add a regular expression check on the client's IP address in Page_Load. If the client's IP is within the range of those in the local network, redirect to WinLogin.aspx. These clients are the ones that can be authenticated using Windows Authentication.

## To Summarize

What the above will do is have any unauthenticated user redirect to Login.aspx. If the client is within the local network (is logged in to the domain), the user will be redirected to WinLogin.aspx, which will perform forms authentication using the user's Windows identity. The user is then redirected to the protected site.

If the client is not within the local network, the user will be shown a login screen. This login screen performs forms authentication as normal, and the user is redirected to the protected site. This is very nice for those who want integrated authentication and a pretty login page.
