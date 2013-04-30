---
layout: post
title: "Adding Properties to Objective-C Categories"
date: 2012-11-02 13:37
comments: false
categories:
- property
- category
- objective-c
---

### Categories, Properties and Ivars

Today I had to add an extra property to a category on an `NSManagedObject`. The property is a bool used to determine the state of the `UITableViewCell` that is supposed to show info from the MO, so it does not need to be stored anywhere. Because of that fact I decided that a category would be the perfect solution. But adding a property to a categor y gives the following error:

    @synthesize not allowed in a category's implementation.
    
Adding an ivar to a category and implement setter/getter manually is a no-go as well. An attempt will also result in an error:


    Ivars may not be placed in categories.  


At this point the question was: Is it even possible to append a class with a property using a category? Yes! The objective-c runtime has a feature called [associated objects](http://developer.apple.com/library/ios/#documentation/cocoa/conceptual/objectivec/Chapters/ocAssociativeReferences.html), which will _[…] simulate the addition of object instance variables to an existing class_. Long story short, I found <a href="http://www.techpaa.com/2012/04/adding-properties-to-categories-and.html">this blog post</a> which explains what to do and the final code looks like this:</p>

**MyManagedObject+Additions.h**

{% highlight objc %}
@property (getter=isRedefinition) BOOL redefined;
{% endhighlight %}

**MyManagedObject+Additions.m**

{% highlight objc %}
NSString const *redefined_key = @"my.very.unique.key";

- (void)setRedefined:(BOOL)redefined
{
	objc_setAssociatedObject(self, &redefined_key, [NSNumber numberWithBool:redefined], OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (BOOL)isRedefinition
{
	return [objc_getAssociatedObject(self, &redefined_key) boolValue];
}
{% endhighlight %}

### Updates

#### 12/03/2012

Another great article about associated objects / references is now available at the [Carbon Emitter blog](http://blog.carbonfive.com/2012/11/27/monkey-patching-ios-with-objective-c-categories-part-ii-adding-instance-properties/).

#### 11/12/2012

The article [Advanced categories with associated objects](http://www.merowing.info/2012/02/associated-objects-for-more-advanced-categories/) by Krysztof Zablocki digs a little deeper with categories and associated objects. It's absolutely worth a read.
