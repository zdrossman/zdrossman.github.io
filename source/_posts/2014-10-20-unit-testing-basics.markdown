---
layout: post
title: "Unit Testing Basics"
date: 2014-10-20 11:39:16 -0400
comments: true
categories: 
---

#Unit testing

## What Problem Are We Solving?

Ahh testing. Here's a topic that warrants an entire book. Or bookshelf.

As a programmer, we are often in the weeds, writing new code, and more frequently editing existing code for that new feature set, bug we found yesterday, or requested changes from our designer. Whatever the case, it can be a challenge to take a step back and think about the requirements of our projects, and how best to implement the logic we are attempting to type in our text editors.

Additionally, if we make a change to a code base, how do we know for sure we haven't upset those requirements, and / or various dependencies on our code?


## What is Unit Testing?

Unit testing is the first line of defense against creating more headaches for ourselves as developers than we were initially trying to solve for.

When we say unit testing, we mean verifying specific units of our code are working as expected. Traditionally in Objective-C, by unit we mean a single Class.


## How to Unit Test

#### Prerequisite: Toolkit

In order to properly unit test in Objective-C, we must first choose our weapon -- the testing spec library we will use to write our tests. For the sake of this blog post (and what I generally use) we will use the combination of [Specta](https://github.com/specta/specta) and [Expecta](https://github.com/specta/expecta). We recommend installing these libraries through [Cocoapods](http://cocoapods.org/).

Once installed, we also recommend installing the [Specta Spec](https://github.com/luiza-cicone/Specta-Templates-Xcode) via [Alcatraz Package Manager](http://alcatraz.io/).

You can then access this template via the File menu, by clicking on "New File" and then choosing "Specta Templates" in the left hand panel.

![](https://github.com/zdrossman/blob/master/spectaTemplateScreenshot.png)

#### Prerequisite: File Setup

Upon opening the Specta Template, there are only a few steps in order to get up and running (once you have added the Specta and Expecta libraries to your Podfile.) Namely, the top of your spec should look like this:

```objc
#import "Specta.h"
#import <The file name of the class you are testing>
#define EXP_SHORTHAND
#import "Expecta.h"

SpecBegin(<The class you are testing goes here, without the extension>)
```

Note: It is important that you define `EXP_SHORTHAND` before importing the `Expecta.h` file in the code.


#### How to Write Useful Tests

The following is a slightly modified example of unit testing from one of our labs:

```objc

#import "Specta.h"
#import "Pet.h"
#define EXP_SHORTHAND
#import "Expecta.h"

SpecBegin(Pet)

describe(@"Pet", ^{

describe(@"makeASound", ^{
it(@"should be an instance method", ^{
Pet *cutePet = [[Pet alloc] init];
expect(cutePet).to.respondTo(@selector(makeASound));
});

it (@"should return the appropriate NSString", ^{
Pet *cutePet = [[Pet alloc] init];
expect([cutePet makeASound]).to.equal(@"Pet me!");
});
});

describe(@"eatSomething", ^{

it (@"should be an instance method", ^{
Pet *cutePet = [[Pet alloc] init];
expect(cutePet).to.respondTo(@selector(eatSomething));
});

it (@"should return the appropriate NSString", ^{
Pet *cutePet = [[Pet alloc] init];
expect([cutePet eatSomething]).to.equal(@"Nom nom nom.");
});

});
});

SpecEnd

```

We'll describe how these tests work, line by line.

Tests are developed as a set of nested blocks. If you haven't yet learned about blocks, all you need to know for now is that they begin with `^{` and end with `}`. For the sake of testing, you can think of blocks as simply a way to delineate sections of code into sub-sections. (There is a lot more to blocks, but for another day.)

Unit testing combines two types of blocks. These are `describe` blocks and `it` blocks. Think of testing as a form of outline. Every `describe` block may contain a `describe` block within it, but eventually it must end in it's innermost block being an `it` block (or multiple `it` blocks). The `it` block actually performs a test. `Describe` blocks simply allow us to build meaningful descriptions (surprise!) of the tests we are about to perform.

So at the outermost level of our unit tests, we begin with a describe block that describes the class we will be testing. In our example, that will be `Pet`.

Within that block, we create additional describe blocks. Our first describes `makeASound`, a method we want to create in our `Pet` class. We can see `makeASound` should be an instance method from the first `it` block, and that it should return an `NSString`, based on the second `it` block.

The language used to test within an `it` block begins with the word `expect`, and takes an argument for the thing we want to test. In our first `it` block, we are expecting the object `cutePet` to `respondTo` the selector (aka method) `makeASound`. By implementing this test, if our class does not include this method, we will get notified.

Note: `it` blocks should generally only contain a single test, i.e. `expect` statement.

Additional note: For a method with arguments, the selector will include the `:` but not the argument names. For instance, if I had the following method...

```
- (void)doSomethingWithName:(NSString *)name andBirthday:(NSDate *)birthdate
```

...the selector would be `@selector(doSomethingWithName:AndBirthday:)`.

The cool thing about this test is that by using these nested `describe` and `it` blocks, when we run our tests the first test will read: "Pet makeASound should be an instance method." Our test results can now be read in plain english.

In our second test, we `expect` that the return value of the method `makeASound` will be an `NSString` and that `NSString` will have the value `@"Pet me!"''

The second set of tests is very similar. For more examples, check out the next section: Standard Types of Unit Tests.

But before we look at other tests, we should think about how these tests might be written more concisely. For instance, you'll notice that in every `it` block we initialize an `Pet` in order to test that class's instance methods. Wouldn't it be nice if we could just write that initialization statement once?

Well, as you probably noted if you have opened the Specta Template already, there are in fact, four blocks we have not talked about yet. These are your `beforeAll`, `beforeEach`, `afterEach`, and `afterAll` blocks. These blocks allow us to run a piece of code either, once (`beforeAll`) prior to the tests running, or prior to each test (`beforeEach`), and similarly following each test being run (`afterEach`) and every test being run (`afterAll`). So a better way to write the code above would be the following:

```objc

#import "Specta.h"
#import "Pet.h"
#define EXP_SHORTHAND
#import "Expecta.h"

SpecBegin(Pet)

describe(@"Pet", ^{

__block Pet *cutePet;

beforeAll(^{

cutePet = [[Pet alloc] init];

});

describe(@"makeASound", ^{
it(@"should be an instance method", ^{
expect(cutePet).to.respondTo(@selector(makeASound));
});

it (@"should return the appropriate NSString", ^{
expect([cutePet makeASound]).to.equal(@"Pet me!");
});
});

describe(@"eatSomething", ^{

it (@"should be an instance method", ^{
expect(cutePet).to.respondTo(@selector(eatSomething));
});

it (@"should return the appropriate NSString", ^{
expect([cutePet eatSomething]).to.equal(@"Nom nom nom.");
});

});
});

SpecEnd
```

You'll notice in the above, I have declared my variable `cutePet` outside of the `beforeAll` block. This is because as with any variable, we need to make sure it has the proper scope to be used elsewhere in our code. This is no different than initializing a variable inside of an `if` statement, where we would only have access to it between the `if` statement's brackets. 

That said, you are probably wondering why I have written `__block` prior to declaring the variable. 

For safety, and this will be further elaborated upon in a lesson on blocks, variables used within a block must be designated this way if they are to be written to. (Note: If you do not use the `__block` syntax and attempt to set a variable inside of a block to a new value, that value will only persist while inside of the block.) In short, expect to be using `__block` syntax for all variables that will be used across various tests.

Now back to our regularly scheduled program...

One last thing before moving on to standard unit test examples. Let's make sure we're all clear on when to use `beforeAll` / `afterAll` vs. `beforeEach` / `afterEach`. Using `all` means that if we initialize a variable there, and we make changes to this variable in the tests, those changes will persist. `beforeEach` is more like having a new instance in each test, as we did earlier in this tutorial.




#### Standard Unit Tests (Source: [Expecta](https://github.com/specta/expecta) )


##### Commonly Used Matchers

###### isEqual

```objc

// compares objects or primitives x and y and passes if they are identical (==) 
// or equivalent (isEqual:).
expect(x).to.equal(y); 

//Examples
expect([@2 integerValue] + [@3 integerValue]).to.equal([@5 integerValue]);
expect([myObject description]).to.equal(@"This is my object");
```

Something to keep in mind about:

```objc
- (BOOL)isEqual:(id)other
```

The `isEqual` matcher does one of two things; either it checks to see if they are `==` the same (as in the same memory address), or that the objects evaluate to the same value (e.g. `@5 = @5`). If you are trying to use `isEqual` with custom objects, however, you must override the `isEqual` method of `NSObject` so that the matcher knows how to evaluate the equivalency.

For instance, if you had an Car object, you might want to check in your `isEqual` override that the model, year, and color are all the same, and only then return YES, to determine equality.

###### beNil


```objc
// passes if x is nil.
expect(x).to.beNil(); 

//Examples
expect([myObject removeFromDataSet]).to.beNil();
```


###### beTruthy / beFalsy

```objc
// passes if x evaluates to true (non-zero).
expect(x).to.beTruthy(); 

// passes if x evaluates to false (zero).
expect(x).to.beFalsy(); 

//Examples
expect([Dog canPurr]).to.beFalsy();
expect([Cat canPurr]).to.beTruthy();

```

###### beInstanceOf

```objc
// passes if x is an instance of a class Foo.
expect(x).to.beInstanceOf([Foo class]); 

//Examples
Dog *dog = [Dog alloc] init]
expect(dog).to.beInstanceOf([Dog class]);
```

###### beKindOf


```objc
// passes if x is an instance of a class Foo or if x is an instance of any class 
// that inherits from the class Foo.
expect(x).to.beKindOf([Foo class]); 

//Examples
Dog *dog = [Dog alloc] init]
expect(dog).to.beKindOf([Pet class]);
```

###### beginWith / startWith / endWith

```objc
// passes if an instance of NSString, NSArray, or NSOrderedSet x begins with y.
// also aliased by startWith
// the expect argument must be of the same type as the beginWith argument
expect(x).to.beginWith(y); 

// passes if an instance of NSString, NSArray, or NSOrderedSet x ends with y.
// the expect argument must be of the same type as the endWith argument
expect(x).to.endWith(y); 

//Examples
NSString *myName = @"Zach"
expect(myName).to.beginWith(@"Za");
expect(myName).to.endWith(@"ch");

NSArray *myFriends = @[@"Joe",@"Chris",@"Al"];
expect(myFriends).to.beginWith(@[@"Joe"]);
expect(myFriends).to.endWith(@[@"Al"]);
```

###### beIdenticalTo

```objc
// compares objects x and y and passes if they are identical and have the 
// same memory address.

expect(x).to.beIdenticalTo(y); 

// Examples:
Dog *dog = [Dog alloc] init];
Pet *pet = (Pet *)dog;

expect(dog).to.beIdenticalTo(pet);
```

###### contain

```objc
// passes if an instance of NSArray or NSString x contains y.
expect(x).to.contain(y);

// Examples:

expect(@"This is my name").to.contain(@"is ");

NSArray *myNameSentence = @[@"This",@"is",@"my",@"name"];
expect(myNameSentence).to.contain(@"is");

```

##### Less commonly used matchers, but still interesting

```objc

// passes if an instance of NSArray, NSSet, NSDictionary or NSOrderedSet x 
// contains all elements of y.
expect(x).to.beSupersetOf(y); 

// passes if an instance of NSArray, NSSet, NSDictionary or NSString x has 
// a count or length of y.
expect(x).to.haveCountOf(y); 

// passes if an instance of NSArray, NSSet, NSDictionary or NSString x has 
// a count or length of 0.
expect(x).to.beEmpty(); 

// passes if the class Foo is a subclass of the class Bar or if it is identical to 
// the class Bar. 
// Use beKindOf() for class clusters.
expect([Foo class]).to.beSubclassOf([Bar class]); 

// passes if x is less than y.
expect(x).to.beLessThan(y); 

// passes if x is less than or equal to y.
expect(x).to.beLessThanOrEqualTo(y); 

// passes if x is greater than y.
expect(x).to.beGreaterThan(y); 

// passes if x is greater than or equal to y.
expect(x).to.beGreaterThanOrEqualTo(y); 

// passes if x is in the range of y and z.
expect(x).to.beInTheRangeOf(y,z); 

// passes if x is close to y.
expect(x).to.beCloseTo(y); 

// passes if x is close to y within z.
expect(x).to.beCloseToWithin(y, z); 

// passes if a given block of code raises an exception named ExceptionName.
expect(^{ /* code */ }).to.raise(@"ExceptionName"); 

// passes if a given block of code raises any exception.
expect(^{ /* code */ }).to.raiseAny(); 

// passes if x conforms to the protocol y.
expect(x).to.conformTo(y); 

// passes if x responds to the selector y.
expect(x).to.respondTo(y); 

// passes if a given block of code generates an NSNotification named NotificationName.
expect(^{ /* code */ }).to.notify(@"NotificationName"); 

// passes if a given block of code generates an NSNotification equal to the passed 
// notification.
expect(^{ /* code */ }).to.notify(notification); 

```

## Unexpected Behaviors

- If you forget to place `()` after the matcher (e.g. `to.beNil` vs. `to.beNil()` ), your test will simply not evaluate. It will appear to pass even if it is failing. Be sure to include your parentheses!

- Tests will be ignored if not written appropriately inside of an `it` block. So you might think they are passing when they are effectively not even part of the tests. Make sure all tests are written inside `it` blocks!

- Tests do not run in a specified order. This is particularly an issue because you might think that you can use a `beforeAll` block when they need a `beforeEach` block, simply because their tests run in an order that affects the state of the variable you created before the tests began in a way you are not expecting. Make sure you consider the state of the program for each test independently from every other.


## Common Error Messages

- We've all done this one before: Opening the .xcodeproj instead of the .xcworkspace and attempting to test. Your testing frameworks will not be a part of the project. You will get an `Apple Mach-O Linker Error` and in the detail see something that says `library not found for -lPods...` This is a good sign that all you have to do is close down the xcodeproj and open up the xcworkspace and you'll be back in business!
