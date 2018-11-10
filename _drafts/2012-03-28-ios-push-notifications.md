---
layout:             post
title:              iOS Push Notifications
date:               2012-03-28 18:38:00 -0400
tags:               
---

Push notifications have been a mystery to me until now, and honestly I was afraid it would be too much work just to experiment with them. The truth is that they're actually really easy. As usual with iOS development, there is a little bit of legwork involved with certificates and such, but following these instructions makes it relatively easy.

These instructions assume you already have a working project with development certification and provisioning profile set up.

## How Push Notifications Work
The Apple Push Notification Service (APNS) is the mediator for all push notifications. It's how Apple guarantees both the source and the destination of push notifications. Here's how it's set up.

The developer needs an SSL certificate to communicate with APNS. Each app needs to be individually configured to send push notifications, and each app gets its own SSL certificate. This is how the source of a notification is guaranteed.

Then the app needs to register with the device for push notifications. When the app registers, it asks APNS for a device token, which will be used to guarantee the destination of each push notification. When the app receives this token, it needs to send it back to the provider so that the provider knows where to send these notifications.

<div style="clear: both; text-align: center;"><a href="http://developer.apple.com/library/mac/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/token_generation.jpg" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="184" src="http://developer.apple.com/library/mac/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/token_generation.jpg" width="320" /></a></div>

When it comes time for a new notification, the provider makes a connection with APNS using its SSL certificate to secure the connection. The provider then sends the notification to APNS identifying the destination using the device token it received earlier. APNS then sends the notification on to the device.

<div style="clear: both; text-align: center;"><a href="http://developer.apple.com/library/mac/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/token_trust.jpg" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="234" src="http://developer.apple.com/library/mac/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/token_trust.jpg" width="320" /></a></div>

<b><span style="font-size: large;">The Certificates</span></b>
First, you need your App ID to be configured for push notifications. Log in to the iOS Dev Center, open up the Provisioning Portal, and go to the App IDs section. You will see this for your app:

<div style="clear: both; text-align: center;"><a href="http://2.bp.blogspot.com/-jjDfQmBDYmQ/T3Mo0x0rMpI/AAAAAAAAEsE/sS0UhqcqpBM/s1600/AppIDConfigurableForPush.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="28" src="http://2.bp.blogspot.com/-jjDfQmBDYmQ/T3Mo0x0rMpI/AAAAAAAAEsE/sS0UhqcqpBM/s320/AppIDConfigurableForPush.png" width="320" /></a></div>
 Click the Configure link. You will get this screen:

<div style="clear: both; text-align: center;"><a href="http://3.bp.blogspot.com/-yibrOK3M3p8/T3Mo1QCKRAI/AAAAAAAAEsQ/cvyggVlPlAE/s1600/EnablePush.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="93" src="http://3.bp.blogspot.com/-yibrOK3M3p8/T3Mo1QCKRAI/AAAAAAAAEsQ/cvyggVlPlAE/s320/EnablePush.png" width="320" /></a></div><div style="clear: both; text-align: -webkit-auto;">
</div><div style="clear: both; text-align: -webkit-auto;">Check the box next to "Enable for Apple Push Notification service" and click the Configure button for "Development Push SSL Certificate". Follow the instructions in the dialog to create and upload a certificate signing request (CSR). Apple will create and sign your SSL certificate, which you can download at the end of the process.</div><div style="clear: both; text-align: -webkit-auto;">
</div><div style="clear: both; text-align: -webkit-auto;"><i>Note: When creating your certificate signing request, make the Common Name something descriptive, like "My App Push Notifications". This way it's easy to differentiate it in your Keychain.</i></div><div style="clear: both; text-align: -webkit-auto;"><i>
</i></div><div style="clear: both; text-align: -webkit-auto;">Store the SSL certificate someplace safe. In Keychain Access, Ctrl-Click the private key that was created with your CSR and export the key to a file. Store it with the SSL certificate. Put the CSR there too, so you can use it to easily renew your SSL certificate before it expires.</div><div style="clear: both; text-align: -webkit-auto;">
</div><div style="clear: both; text-align: -webkit-auto;">Now you have the certificates you need for your provider server to send push notifications. For the push service example below, we'll need to convert the certificates. First, convert the private key from a .p12 file to a .pem file:</div><div style="clear: both; text-align: -webkit-auto;">
</div><div style="clear: both; text-align: -webkit-auto;"></div>
```
$ openssl pkcs12 -nocerts -in PrivateKey.p12 -out PrivateKey.pem
```
<div style="clear: both; text-align: -webkit-auto;">
</div><div style="clear: both; text-align: -webkit-auto;">Be sure to enter a PEM pass phrase when prompted. It may not actually work if you don't. Next, convert the SSL certificate to a .pem file also:</div><div style="clear: both; text-align: -webkit-auto;">
</div><div style="clear: both; text-align: -webkit-auto;"></div>
```
$ openssl x509 -in aps_development.cer -inform der -out PublicCertificate.pem
```
<div style="clear: both; text-align: -webkit-auto;">
</div><div style="clear: both; text-align: -webkit-auto;">Then, combine the two in a single file for the service to use. Be sure you put the SSL certificate before the private key:</div><div style="clear: both; text-align: -webkit-auto;">
</div><div style="clear: both; text-align: -webkit-auto;"></div>
```
$ cat PublicCertificate.pem PrivateKey.pem &gt; PushNotificationCertificate.pem
```
<b>
</b>
<b><span style="font-size: large;">The App</span></b>
The app needs to register for push notifications before any can be sent. The process is simple enough. Just add something similar to the following code to application:didFinishLaunchingWithOptions: inside of your AppDelegate file:


```
[[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeAlert)];
```

You can use any combination of the notification types if you don't plan on using all three. Even though this line will get called every time the app launches, it will only have any effect the first time. When this line does get called, the user will be presented with a dialog like this:

<div style="clear: both; text-align: center;"></div><div style="clear: both; text-align: center;"><a href="http://1.bp.blogspot.com/-uwSfUc1yoYE/T3NaSzECJsI/AAAAAAAAEsw/Mmajf93h0LI/s1600/RegisterForPush.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://1.bp.blogspot.com/-uwSfUc1yoYE/T3NaSzECJsI/AAAAAAAAEsw/Mmajf93h0LI/s1600/RegisterForPush.png" /></a></div>

If they select "Don't Allow", then the user will need to go into Settings to begin allowing notifications from your app.

Once your app registers itself, it needs to receive its device token. Add the following to your AppDelegate, adding any code necessary to send the token to your server. It might be a good idea to save the token locally, as well, in case you need to contact the server with it again.


```
- (void)application:(UIApplication*)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData*)deviceToken
{
    NSLog("Device token: %@", deviceToken);
    // Save/send token to server
}

- (void)application:(UIApplication*)application didFailToRegisterForRemoteNotificationsWithError:(NSError*)error
{
    NSLog("Failed to register: %@", error);
}
```

The use of NSLog here is simply so we can see the device token. We're going to copy it for use with our makeshift notification server.

Unregistering the device for notifications is as simple as


```
[[UIApplication sharedApplication] unregisterForRemoteNotifications];
```


When a notification is received, the user can open the app directly. In AppDelegate in application:didFinishLaunchingWithOptions:, you can access the notification payload with the following:


```
NSDictionary* remoteNotif = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
```

Also take note that if the app is open when a notification is received, the user won't get the usual notification alert. Instead, the app is notified in the AppDelegate via


```
- (void)application:(UIApplication*)application didReceiveRemoteNotification:(NSDictionary*)userInfo
```

You can take this opportunity to show the user an alert or simply process the notification yourself.

<b><span style="font-size: large;">The Provider Server</span></b>
As far as sending a single notification goes, the server's job is pretty simple. Just establish a secure connection with APNS using the SSL certificate we made earlier and send the notification. Clearly, a fully-functional and useful server is going to be more complicated, but this example is going to show only the actual sending of a notification.

What this example does not do (because it is out of scope) is accept device tokens from devices, keep track of what badge numbers should be, schedule notification sending, and send multiple notifications to multiple devices.

I set up a simple ruby environment with the APNS gem installed (gem install apns). With that done, here is the script I use for simple testing:


```
require 'apns'

APNS.host = 'gateway.sandbox.push.apple.com'
APNS.pem = 'PushNotificationCertificate.pem'
APNS.pass = 'privateKeyPassword'
APNS.port = 2195
device_token = 'a92842700f3accfa59bcb844b6f1a051111493d4d07ab3bfa3ba34237a0d3031'

APNS.send_notification(device_token, :alert =&gt; 'This is a notification', :badge =&gt; 0, :sound =&gt; 'default')
```
<div>
</div><div>APNS.host is 'gateway.sandbox.push.apple.com' for development testing. It's 'gateway.push.apple.com' for production. Be sure to remove spaces and angle brackets from the device token. APNS.password is the password you used with your private key earlier. I got device_token from the NSLog after running the app with the code to register for notifications.

Simply running this script with the correct device_token and SSL certificate will show a notification on your iOS device with the message "This is a notification". The user can then open your app by selecting the notification.</div><b>
</b>
<b><span style="font-size: large;">Anatomy of a Push Notification</span></b>
A push notification is nothing more than a JSON object sent to APNS. This JSON object looks similar to this:


```
{
    "aps":{
        "alert":"This is a notification message",
        "badge":1,
        "sound":"default"
    },
    "some-custom-data":{
        "data":"Whatever I need goes here"
    }
    }
```

The "aps" item contains the stuff important to a push notification. Only the relevant items need be set. If your app isn't registered for badges, there's no need to set "badge". In addition to "aps", you can also set some custom data to send to your app if the user opens the app via the notification (or if the app is already open). This data can be accessed via the code shown in "The App" section above.

The alert item can contain a simple message to show to the user as shown here, or it can be more complex. It can be an object containing a message and localization info. See <a href="http://developer.apple.com/library/mac/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/ApplePushService/ApplePushService.html" target="_blank">Apple's Documentation</a> on push notifications for more info.
<b>
</b>
<b><span style="font-size: large;">A few things to keep in mind</span></b>
Push notifications are NOT guaranteed to reach their destination. If the iOS device is turned off at the time, APNS will keep track of the <i>most recent notification </i>to be sent. It will try to send that notification, but will give up after some time if delivery remains unsuccessful. This means that if you were to create, say, a chat app, you could not rely on the push notifications to send the entire chat. Instead, use the push notification to notify the user of recent chat activity, but poll the server for the full chat once the app is opened.

Push notifications are NOT necessarily instant. They could take anywhere from a second to a half hour. This could be due to high network traffic, an overloaded APNS, or other unknown causes.

There is NO WAY to determine whether the device actually received the notification. Sure, you can program your app to let the server know that it received the notification, but that will only work if the user opens the app with the notification. If the user clears it, it's gone.

Do not send sensitive information via push notification! Push notifications are meant simply to notify the user of something, whether it's that some information is available or that something or someone is trying to communicate with you.

<b><span style="font-size: large;">Final Notes</span></b>
Simple push notifications can easily be set up in a day. Extra development time comes in the form of using the notification to do something special within the app and in developing a server that sends out notifications on a regular basis.

See <a href="http://developer.apple.com/library/mac/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Introduction/Introduction.html" target="_blank">Apple's Documentation</a> regarding push notifications for more information.


