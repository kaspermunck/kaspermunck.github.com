---
layout: post
title: "Unit Testing Singletons"
date: 2013-05-22 13:37
comments: true
categories:
- unit testing
- singleton
- sharedInstance
- KMSwizzleton
---

Whether to use singletons or not is a question of religion and I won't be judging anyone. In my opinion there are certain situations where singletons just make our lives easier, hence I introduce a singleton every now and then, when appropriate. However, testing singletons is no easy task, which is why I tend to avoid it.

In an attempt to make things easier, I created [KMSwizzleton](https://github.com/kaspermunck/KMSwizzleton), a helper class that makes the following possible:

{% highlight objc %}
id fakeSingleton = [OCMockObject mockForClass:[MySingleton class]]; // or any other mock library
[KMSwizzleton stubSingleton:[MySingleton class] andReturnFakeInstance:fakeSingleton];
id instance = [MySingleton sharedInstance]; // returns fakeSingleton
{% endhighlight %}

This makes it possible to inject a stub or mock singleton into a closed method, which otherwise would quite a hassle. It also gets rid of the need to expose a property injection method or to override `+sharedInstance` in a testable subclass of `MySingleton`. After `+sharedInstance` is called once, `MySingleton` is reverted to its original state.