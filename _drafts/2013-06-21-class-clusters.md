---
layout: post
title: "put title here"
date: 2012-16-21 13:37
comments: false
categories:
- ios6
- ios7
- class cluster
- ui support
---

### Transitioning to iOS 7

With iOS 7 Apple wants iOS deve

[iOS 7 Transition Guide](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/TransitionGuide/index.html#//apple_ref/doc/uid/TP40013174)

**The message is clear:**
> Focus on redesigning the app for iOS 7 first. Then, bring the changes to the iOS 6 version as appropriate

### Class Clusters

If you've ever tried to subclass NSDictionary (or NSArray ), for example to 

````
'*** -[NSDictionary objectForKey:]: method only defined for abstract class.  Define -[MyDictionary objectForKey:]!'
````

[NSDictionary](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSDictionary_Class/Reference/Reference.html):
> If you do need to subclass NSDictionary, you need to take into account that is represented by a Class cluster.

This means that with `[[NSDictionary alloc] init]` you don't get an instance of NSDictionary itself, but instead one of its derivatives. More specifically, a concrete implementation of NSDictionary that will suit your needs best.

[Class Cluster](http://developer.apple.com/library/ios/#documentation/general/conceptual/devpedia-cocoacore/ClassCluster.html):
> A class cluster is an architecture that groups a number of private, concrete subclasses under a public, abstract superclass. The grouping of classes in this way provides a simplified interface to the user, who sees only the publicly visible architecture.

This is a clever design choice, and it's a technique that can be taken advantage of when dealing with UI specific to a version of the OS, here iOS 6 vs iOS 7.

### The UIButton Class Cluster

To take advantage of class clusters in this scenario, we'll create a UIButton subclass, `Button`. This, when allocated, allocates an instance of `IOS7Button` when compiled for iOS 7 and `IOS6Button` when compiled for iOS 6, respectively. The magic is performed inside `Button` so that users of it will not need to worry about boring stuff like OS version.

````
@interface Button : UIButton

@end

…

@interface IOS7Button : Button
…

@interface IOS6Button : Button
…
````

**Suggested by Apple in iOS 7 Transition Guide:**
There are several ways to detect version of the OS currently installed on a device. This sample is from iOS 7 Transition Guide:

````
NSUInteger DeviceSystemMajorVersion();NSUInteger DeviceSystemMajorVersion() {    static NSUInteger _deviceSystemMajorVersion = -1;
    static dispatch_once_t onceToken;    dispatch_once(&onceToken, ^{        _deviceSystemMajorVersion = [[[[[UIDevice currentDevice] systemVersion]                                       componentsSeparatedByString:@"."] objectAtIndex:0] intValue];    });    return _deviceSystemMajorVersion;}#define IS_IOS_7 (DeviceSystemMajorVersion() < 7)
````

Then, in Button.m, override `+alloc`. This is where we will make the decistion about which class in the cluster should actually be used.

````
+ (id)alloc {
    // avoid recursive invocation of +alloc from subclasses
    if( [self class] == [Button class])
    {
        if(IS_IOS_7 >= 7)
        {
            return [IOS7Button alloc];
        }
        else
        {
            return [IOS6Button alloc];
        }
    }
    else
    {
        return [super alloc];
    }
}
````

Without the check of class, sending `+alloc` to one of the subclasses will result in alloc calling itself recursively until the app crashes. Remember, IOS7- and IOS6Button inherit this behavior from Button.

That's it, try changing the look of the buttons and create a Button-instance:

````
Button *button = [[Button alloc] init];
````

If you've already installed Xcode 5 and the iOS7 beta SDK, build for both and note the difference. If not, you can cheat and force IS_IOS_7 to return 7, just to be able to see the difference.

### Making Button 'abstract'

The concept of abstract classes does not exist in Objective-C. However, it's opssible to mimic abstract behavior to some degree. Add a dummy method to Button, such as:

````
- (void)invokeSpecialBehavior;
````

This is a method that should behave differently in version of iOS prior to 7 and 7; let's say that it should use UI Dynamics [xyz xyz xyz], and normal UIView animations in iOS 6.

````
- (void)invokeSpecialBehavior {
    
    NSAssert(NO, @"subclasses of %@ must implement abstract method %@.", NSStringFromClass([self superclass]), NSStringFromSelector(_cmd));
}
````

Now, call `invokeSpecialBehavior` on the Button we allocated before. This will result in an exception, telling us that we must implement `invokeSpecialBehavior` in our subclasses:

**abstract methods in base class**:
>'NSInternalInconsistencyException', reason: 'subclasses of Button must implement abstract method invokeSpecialBehavior.'