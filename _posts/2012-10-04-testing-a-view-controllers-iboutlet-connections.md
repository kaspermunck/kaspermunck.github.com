---
layout: post
title: "Testing a View Controller's IBOutlet Connections"
date: 2012-10-04 13:37
comments: false
categories:
- unit test
- viewcontroller
- iboutlet
---

### When Using Nib Files

Understanding how a ViewController's view hierarchy is loaded is essential when testing UI related stuff. Consider the following test case, which verifies that an IBOutlet connection to `myLabel` has been established when a view controller is loaded:

{% highlight objc %}
MyViewController *vc = [[MyViewController alloc] initWithNibName:@"MyViewController" bundle:nil];
STAssertNotNil(vc.myLabel, @"Should connect myLabel IBOutlet.");
{% endhighlight %}

The test will always fail, telling you that `vc.myLabel` is nil. There is a very good reason for that. [Apple's View Controller Programmer Guide for iOS](http://developer.apple.com/library/ios/#featuredarticles/ViewControllerPGforiPhoneOS/ViewLoadingandUnloading/ViewLoadingandUnloading.html) illustrates a view controller's loading and unloading sequence with the following diagram: 

<img class="align-center" title="Instantiating View Hierarchy" src="http://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Art/loading_a_view_into_memory_2x.png" width="355" height="302" />

It is described as follows: 

> Whenever some part of your app asks the view controller for its view object and that object is not currently in memory, the view controller loads the view hierarchy into memory and stores it in its view property for future reference.

This explains why `myLabel` is always nil in the test case above; it has not yet been loaded, since - at the time `STAssertNotNil` is reached - the view property has not yet been accessed. To force loading of the view hierarchy, invoke the `view` property of the viewcontroller:

{% highlight objc %}
MyViewController *vc = [[MyViewController alloc] initWithNibName:@"MyViewController" bundle:nil];
[vc view]; // load view hierarchy (if not already loaded)
STAssertNotNil(vc.myLabel, @"Should connect myLabel IBOutlet.");
{% endhighlight %}

Now, as opposed to before, the test passes, because _the view controller loads the view hierarchy into memory and stores it in its view property_ 

### When using Storyboards

When using storyboards, initializing a viewcontroller under test with `initWithNibName:bundle:` or `init` no longer makes sense, since the views are now embedded in a storyboard, hence no longer accessible via a nib file. To be able to load a viewcontroller realistically when using storyboards, follow these two steps:

- Assign an identifier string to the viewcontroller in the storyboard.
- Load the storyboard programmatically and ask it for a viewcontroller with a desired identifier.

Assume a storboard containing the view of `MyViewController`. First, in the Storyboard, select `MyViewController` and, in Utilities View, select the Identity Inspector. In here you will find an Identity Setting named `Storyboard ID`. This is used to identify the viewcontroller. Assign a unique, descriptive value, eg `MyViewController`.

![Storyboard viewcontroller identifier](/images/storyboard-identifier.png)

Now, when initializing `MyViewController` from test code, follow this pattern to load it programmatically (where _Storyboard_ is the name of the containing storyboard file):

{% highlight objc %}
UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Storyboard" bundle:nil];
MyViewController *vc = [storyboard instantiateViewControllerWithIdentifier:@"MyViewController"];
{% endhighlight %}

With this approach you don't have to invoke the view property, since the view hierarchy has already been loaded into memory at this time.
