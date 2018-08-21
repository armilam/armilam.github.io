---
layout:             post
title:              Android for iOS Developers - Activities
date:               2014-04-07 16:21:00 -400
tags:               android ios
---

I began learning iOS development a few years ago. As one of the few developers at my company who knew how to write iOS apps, I quickly gained a lot of experience. When I was asked recently to help on the Android version of an app I wrote for a client, I found myself struggling to change my mental models for the architectures behind the development tools and language for the OS.

Now that I've got a better grasp on everything, I've decided to write a series of instructional blog posts for those who know how to do things in iOS and are trying to figure out how to translate those skills to Android.

### Activities

In the land of iOS, we're used to using UIViewControllers that are responsible for manipulating the UI. Each view controller is generally responsible for one screen and all views contained in that screen. UIViewController has a lifecycle for managing when the view is shown, hidden, created, destroyed, running out of memory, and much more.

In Android, we deal with the <a href="http://developer.android.com/reference/android/app/Activity.html" target="_blank">Activity</a> class. Right off the bat, here are some things you should know about it.

### An Activity essentially represents a screen (in the user's point of view)

A UIViewController is often used in this way, but it does not have to be. A UIViewController can be a child of another UIViewController, however this is not possible in Android. Instead, we use <a href="http://developer.android.com/reference/android/app/Fragment.html" target="_blank">Fragments</a>. More on that in another post.

### It has a lifecycle much like UIViewController

The reference at <a href="http://developer.android.com/reference/android/app/Activity.html" target="_blank">developer.android.com</a> has more detail about this. Here are the basics:
1. onCreate
    - Called after the Activity object is created. You'll usually do any initial UI setup here.
1. onResume
    - Called after onCreate, but more importantly it's also called when the Activity returns to the foreground after having been paused. Resume anything you paused in onPause or check for updates that you may want to use upon returning to this Activity.
1. onPause
    - Called when another Activity comes into the foreground. This is a good place to save the user's work, pause any tasks in progress, etc.
1. onStop
    - The OS is going to destroy the Activity object.
1. onSaveInstanceState
    - This is a good place to save state because this Activity going to be destroyed or recreated. It is important to know that simply rotating the device orientation will cause the Activity to be recreated. If there is text in any TextViews or data loaded from a web service, you can save that here to keep from losing the user's work or having to call the web service again.
1. onRestoreInstanceState
    - After the Activity has been recreated, restore state here. Any data you save in onSaveInstanceState you'll be able to retrieve here.

### Similar to loading a nib with a UIViewController, you'll specify a UI file to load with an Activity

In iOS we use xib or storyboard files. In Android, it's layout files defined in XML. I'll write more on the specifics of Android XML files in another post.

What you need to know right away is that IDs and other resources from XML files are automatically compiled into constants in R.java. This is how we can easily refer to XML objects. You can see that in action below, where we identify the UI file to load with an activity:
{% highlight java %}
@Override
public void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.my_layout);
}
{% endhighlight %}

Later, when we need a reference to a specific view within the Activity, we can find it by its ID:
{% highlight java %}
TextView textView = (TextView) findViewById(R.id.my_text_view);
{% endhighlight %}

### Unlike with UIViewController, you likely won't instantiate Activities directly

Instead, you'll call startActivity with an <a href="http://developer.android.com/reference/android/content/Intent.html" target="_blank">Intent</a> to start a specific Activity class. UI creation is handled in the Activity lifecycle. To switch to a new Activity called MyActivity, we must first define it in AndroidManifest.xml:

{% highlight java %}
<activity android:name="com.aaron-milam.MyActivity"></activity>
{% endhighlight %}

Then we can call an Intent to start an instance of MyActivity:

{% highlight java %}
Intent myIntent = new Intent(MyActivity.class);
startActivity(myIntent);
{% endhighlight %}

Because of this, there's no need for anything like UINavigationController, especially if you're making use of <a href="http://developer.android.com/reference/android/support/v7/app/ActionBarActivity.html" target="_blank">ActionBarActivity</a>.
