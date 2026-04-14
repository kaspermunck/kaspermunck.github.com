---
layout: post
title: "Adding Properties to Objective-C Categories"
date: 2012-11-02
permalink: /2012/11/adding-properties-to-objective-c-categories/
---

### Categories, Properties and Ivars

Sometimes, subclassing causes unneccessary overhead (or is even discouraged by Apple, such as in the case of `NSArray` or `NSDictionary`). Especially if you need no more than a few extra properties. A category appears as a simpler solution, however, adding properties to a category is prohibited. Doing so causes one of the following errors, depending on which version of Xcode you are using:

**No auto synthesis (before Xcode 4.4)**
```
@synthesize not allowed in a category's implementation.
```
**Auto synthesis**
```
Property 'myProperty' requires method 'myProperty/setMyProperty' to be defined [...] 
```
Manually implementing a getter for the property could be the solution. Consider an `NSManagedObject` subclass with `NSString` properties for given and family name, respectively. It is not unlike that you would provide a property for getting the full name:

```objc
// in the interface file
@property (readonly, nonatomic) NSString *fullName;

// in the implementation file
- (NSString *)fullName {
    return [NSString stringWithFormat:@"%@ %@", self.givenName, self.familyName];
}
```

But, creating a setter is not as simple as creating a getter. An ivar for storing the value is required, and adding an ivar to a category will also result in an error:
```
Instance variables may not be placed in categories.  
```
This is due to the fact that it is impossible to guarantee a unique ivar. This could potentially lead to a situation where setting a new value for property A causes the value of property B to change too, because of clashing ivars.

Luckily, the Objective-C runtime has a feature called [associated objects](http://developer.apple.com/library/ios/#documentation/cocoa/conceptual/objectivec/Chapters/ocAssociativeReferences.html), which will *[…] simulate the addition of object instance variables to an existing class*. [This blog post at Techpaa](http://www.techpaa.com/2012/04/adding-properties-to-categories-and.html) explains associated objects in details. In short, you will have to provide an object to associate the "ivar" with, a unique key by which it is identified, and a so-called association policy. This policy must obey the characteristics of the property you add (that is, atomicity, memory management etc.):

**MyManagedObject+Additions.h**

```objc
@property (strong, nonatomic) NSString *test;
```

**MyManagedObject+Additions.m**

```objc
NSString const *key = @"my.very.unique.key";

- (void)setTest:(NSString *)test
{
    objc_setAssociatedObject(self, &key, test, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (NSString *)test
{
    return objc_getAssociatedObject(self, &key);
}
```

### Updates

#### 12/03/2012

A great article about associated objects / references is now available at the [Carbon Emitter blog](http://blog.carbonfive.com/2012/11/27/monkey-patching-ios-with-objective-c-categories-part-ii-adding-instance-properties/).

#### 11/12/2012

The article [Advanced categories with associated objects](http://www.merowing.info/2012/02/associated-objects-for-more-advanced-categories/) by Krysztof Zablocki digs a little deeper with categories and associated objects. Absolutely worth a read.
