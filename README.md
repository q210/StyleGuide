# Objective-C Style Guide

This style is very much work in progress and any thoughts and suggestions are welcome!
This was derived from other great guides:

* [NYTimes Objective-C Style Guide](https://github.com/NYTimes/objective-c-style-guide/blob/master/README.md)
* [GitHub Objective-C Style Guide](https://github.com/github/objective-c-style-guide)
* [Structuring Modern Objective-C](https://ashfurrow.com/blog/structuring-modern-objective-c/)

## Introduction

Here are some of the documents from Apple that informed the style guide. If something isn’t mentioned here, it’s probably covered in great detail in one of these:

* [The Objective-C Programming Language](http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html)
* [Cocoa Fundamentals Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/Introduction.html)
* [Coding Guidelines for Cocoa](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
* [iOS App Programming Guide](http://developer.apple.com/library/ios/#documentation/iphone/conceptual/iphoneosprogrammingguide/Introduction/Introduction.html)

This style guide conforms to IETF's [RFC 2119](http://tools.ietf.org/html/rfc2119). In particular, code which goes against the RECOMMENDED/SHOULD style is allowed, but should be carefully considered.

## Table of Contents

* [Documentation and Organization](#documentation-and-organization)
* [Spacing](#spacing)
* [Conditionals](#conditionals)
  * [Ternary Operator](#ternary-operator)
* [Error handling](#error-handling)
* [Methods](#methods)
* [Types](#types)
* [Blocks](#blocks)
* [Dot Notation Syntax](#dot-notation-syntax)
* [Variables](#variables)
  * [Properties](#properties)
  * [Variable Qualifiers](#variable-qualifiers)
* [Naming](#naming)
  * [Categories](#categories)
* [Comments](#comments)
* [Init & Dealloc](#init-and-dealloc)
* [Literals](#literals)
* [CGRect Functions](#cgrect-functions)
* [Constants](#constants)
* [Enumerated Types](#enumerated-types)
* [Bitmasks](#bitmasks)
* [Case Statements](#case-statements)
* [Loop Statements](#loop-statements)
* [Private Properties](#private-properties)
* [Image Naming](#image-naming)
* [Booleans](#booleans)
* [Singletons](#singletons)
* [Imports](#imports)
* [Protocols](#protocols)
* [Golden Path](#golden-path)
* [Xcode Project](#xcode-project)
* [Commit messages](#commit-messages)
* [Boyscout](#boyscout)


## Documentation and Organization

 * Document non-trivial method declarations.
 * Document whether object parameters allow `nil` as a value when in doubt.
 Use `#pragma mark -` to categorize methods in functional groupings and protocol/delegate implementations following this general structure.

**For example:**
```objc
#pragma mark - Lifecycle

- (instancetype)init {}
- (void)dealloc {}
- (void)viewDidLoad {}
- (void)viewWillAppear:(BOOL)animated {}
- (void)didReceiveMemoryWarning {}

#pragma mark - Custom Accessors

- (void)setCustomProperty:(id)value {}
- (id)customProperty {}

#pragma mark - IBActions

- (IBAction)submitData:(id)sender {}

#pragma mark - Public

- (void)publicMethod {}

#pragma mark - Private

- (void)privateMethod {}

#pragma mark - Protocol conformance
// protocol conformance will be shown in XCode file popover as a block with dividers 
// and concrete protocol names will be inside as labels
#pragma mark UITextFieldDelegate
#pragma mark UITableViewDataSource
#pragma mark UITableViewDelegate

#pragma mark - NSCopying

- (id)copyWithZone:(NSZone *)zone {}

#pragma mark - NSObject

- (NSString *)description {}
```

## Spacing

* Indentation MUST use 4 spaces. Be sure to set auto convertation of tabs to spaces in Xcode (Preferences -> Text Editing -> Indentation).
* Files SHOULD be terminated with newline
* Code MUST NOT have any trailing whitespaces
* Class and Method braces MUST open on the next line after the statement

**For example:**
```objc
@implementation StoreSync
{
    // some instance variables declaration
}

+ (StoreSync*)sharedInstance
{
    static id _singleton = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _singleton = [[StoreSync alloc] init];
    });
    return _singleton;
}
```

* other braces (`if`/`else`/`switch`/`while` etc.) MUST open on the same line as the statement. Braces MUST close on a new line.

**For example:**
```objc
if (user.isHappy) {
    // Do something

} else {
    // Do something else
}
```

* There SHOULD be exactly one blank line between methods to aid in visual clarity and organization.
* Lines SHOULD be no longer than 120 symbols to allow comfortable work in two side-by-side editor windows in XCode. 
* Whitespace within methods MAY separate functionality, though this inclination often indicates an opportunity to split the method into several, smaller methods. In methods with long or verbose names, a single line of whitespace MAY be used to provide visual separation before the method’s body.
* `@synthesize` and `@dynamic` MUST each be declared on new lines in the implementation.

## Conditionals

Conditional bodies MUST use braces even when a conditional body could be written without braces (e.g., it is one line only) to prevent errors when changing this code later. 
It's really easy to overlook lack of curly braces and introduce a bug with the second statement in the `else` branch, adding a second line and expecting it to be part of the if-statement. Another, [even more dangerous defect](http://programmers.stackexchange.com/a/16530). Also errors can happen where the line “inside” the if-statement is commented out, and the next line unwittingly becomes part of the if-statement. In addition, this style is more consistent with all other conditionals, and therefore more easily scannable.

**For example:**
```objc
if (!error) {
    return success;
}
```

**Not:**
```objc
if (!error)
    return success;
```

or

```objc
if (!error) return success;
```

### Ternary Operator

The intent of the ternary operator, `?` , is to increase clarity or code neatness. The ternary SHOULD only evaluate a single condition per expression. Evaluating multiple conditions is usually more understandable as an if statement or refactored into named variables.

**For example:**
```objc
result = a > b ? x : y;
```

**Not:**
```objc
result = a > b ? x = c > d ? c : d : y;
```

## Error Handling


* Don't use exceptions for flow control.
* Use exceptions only to indicate programmer error.
* To indicate errors, use an `NSError **` argument or send an error on a [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) signal.
* When methods return an error parameter by reference, code MUST switch on the returned value and MUST NOT switch on the error variable.

**For example:**
```objc
NSError *error;
if (![self trySomethingWithError:&error]) {
    // Handle Error
}
```

**Not:**
```objc
NSError *error;
[self trySomethingWithError:&error];
if (error) {
    // Handle Error
}
```

Some of Apple’s APIs write garbage values to the error parameter (if non-NULL) in successful cases, so switching on the error can cause false negatives (and subsequently crash).

## Methods

* In method signatures, there SHOULD be a space after the scope (`-` or `+` symbol). 
* There SHOULD be a space between the method segments. 
* There SHOULD be a space between type name and asterisk symbol `*`.
* There SHOULD NOT be any spaces between types and method or parameter names.

**For example:**
```objc
- (void)setExampleText:(NSString *)text image:(UIImage *)image;
```

**Not:**
```objc
-(void)setExampleText:(NSString *)text image:(UIImage *)image;
- (void) setExampleText: (NSString *)text image: (UIImage *)image;
- (void)setExampleText:(NSString*)text image:(UIImage*)image;
```

* C function declarations should have no space before the opening parenthesis, and should be namespaced just like a class.

**For example:**
```objc
void GHAwesomeFunction(BOOL hasSomeArgs);
```

* Constructors should generally return [`instancetype`](http://clang.llvm.org/docs/LanguageExtensions.html#related-result-types) rather than `id`.
* Prefer helper functions (such as `CGRectMake()`) to C99 struct initialiser syntax.

**For example:**
```objc
  CGRect rect = CGRectMake(3.0, 12.0, 15.0, 80.0);
```

* Colon-aligning method invocation should often be avoided.  There are cases where a method signature may have >= 3 colons and colon-aligning makes the code more readable. Please do **NOT** however colon align methods containing blocks because Xcode's indenting makes it illegible.

**For example:**

```objc
// blocks are easily readable
[UIView animateWithDuration:1.0 animations:^{
  // something
} completion:^(BOOL finished) {
  // something
}];
```

**Not:**

```objc
// colon-aligning makes the block indentation hard to read
[UIView animateWithDuration:1.0
                 animations:^{
                     // something
                 }
                 completion:^(BOOL finished) {
                     // something
                 }];
```

## Blocks

 * Blocks should have a space between their return type and name.
 * Block definitions should omit their return type when possible.
 * Block definitions should omit their arguments if they are `void`.
 * Parameters in block types should be named unless the block is initialized immediately.

**For example:**
```objc
void (^blockName1)(void) = ^{
    // do some things
};

id (^blockName2)(id) = ^ id (id args) {
    // do some things
};
```

## Types

`NSInteger` and `NSUInteger` should be used instead of `int`, `long`, etc per Apple's best practices and 64-bit safety. `CGFloat` is preferred over `float` for the same reasons. This future proofs code for 64-bit platforms.

All Apple types should be used over primitive ones. For example, if you are working with time intervals, use `NSTimeInterval` instead of `double` even though it is synonymous. This is considered best practice and makes for clearer code.


## Dot Notation Syntax

Dot notation is RECOMMENDED over bracket notation for getting and setting properties.

**For example:**
```objc
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
```

**Not:**
```objc
[view setBackgroundColor:[UIColor orangeColor]];
UIApplication.sharedApplication.delegate;
```

**Exception:** Brackets notation is acceptable when type casting:
```objc
[(DbImage *)obj image_changed]
```

## Variables

Variables SHOULD be named descriptively, with the variable’s name clearly communicating what the variable _is_ and pertinent information a programmer needs to use that value properly.

**For example:**

* `NSString *title`: It is reasonable to assume a “title” is a string.
* `NSString *titleHTML`: This indicates a title that may contain HTML which needs parsing for display. _“HTML” is needed for a programmer to use this variable effectively._
* `NSAttributedString *titleAttributedString`: A title, already formatted for display. _`AttributedString` hints that this value is not just a vanilla title, and adding it could be a reasonable choice depending on context._
* `NSDate *now`: _No further clarification is needed._
* `NSDate *lastModifiedDate`: Simply `lastModified` can be ambiguous; depending on context, one could reasonably assume it is one of a few different types.
* `NSURL *URL` vs. `NSString *URLString`: In situations when a value can reasonably be represented by different classes, it is often useful to disambiguate in the variable’s name.
* `NSString *releaseDateString`: Another example where a value could be represented by another class, and the name can help disambiguate.

Single letter variable names are NOT RECOMMENDED, except as simple counter variables in loops.

Asterisks indicating a type is a pointer MUST be “attached to” the variable name. **For example,** `NSString *text` **not** `NSString* text` or `NSString * text`, except in the case of constants (`NSString * const NYTConstantString`).

#### Properties

* Property definitions SHOULD be used in place of naked instance variables whenever possible. Direct instance variable access SHOULD be avoided except in initializer methods (`init`, `initWithCoder:`, etc…), `dealloc` methods and within custom setters and getters. For more information, see [Apple’s docs on using accessor methods in initializer methods and `dealloc`](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6).

**For example:**

```objc
@interface NYTSection: NSObject

@property (nonatomic) NSString *headline;

@end

@implementation NYTSection

- (instancetype)init
{
    self = [super init];
    if (self) {
        _headline = @"MARS ATTACKS!";
    }

    return self;
}

- (NSString *)toneDownHeadline
{
    return [NSString stringWithFormat:@"%@, probably", self.headline];
}

@end
```

**Not:**

```objc
@interface NYTSection : NSObject {
    NSString *headline;
}
```

* Prefer exposing an immutable type for a property even if it being mutable is an implementation detail.

**For example:**

```objc
// NYTSection.h
@interface NYTSection: NSObject

@property (nonatomic) NSArray *sections;

@end

// NYTSection.m

@interface NYTSection ()

@property (nonatomic) NSMutableArray *sections;

@end
```

* Always declare memory-management semantics even on `readonly` properties.
* Declare properties `readonly` if they are only set once in `-init`.
* Don't use `@synthesize` unless the compiler requires it. Note that optional properties in protocols must be explicitly synthesized in order to exist.
* Declare properties `copy` if they return immutable objects and aren't ever mutated in the implementation. `strong` should only be used when exposing a mutable object, or an object that does not conform to `<NSCopying
* Avoid `weak` properties whenever possible. A long-lived weak reference is usually a code smell that should be refactored out.
* Don't put a space between an object type and the protocol it conforms to.

**For example:**

```objc
@property (attributes) id<Protocol> object;
@property (nonatomic, strong) NSObject<Protocol> *object;
```

* If the property is `nonatomic`, it should be first. The next option should **always** be `retain` or `assign` since if it is omitted, there is a warning. `readonly` should be the next option if it is specified. `readwrite` should never be specified in header file. `readwrite` should only be used in class extensions. `getter` or `setter` should be last. `setter` should rarely be used.

```objc
@property (nonatomic, retain) UIColor *topColor;
@property (nonatomic, assign) CGSize shadowOffset;
@property (nonatomic, retain, readonly) UIActivityIndicatorView *activityIndicator;
@property (nonatomic, assign, getter=isLoading) BOOL loading;
```


#### Variable Qualifiers

When it comes to the variable qualifiers [introduced with ARC](https://developer.apple.com/library/ios/releasenotes/objectivec/rn-transitioningtoarc/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226-CH1-SW4), the qualifier (`__strong`, `__weak`, `__unsafe_unretained`, `__autoreleasing`) SHOULD be placed before the asterisks and the variable name, e.g., `__weak NSString * text`. 

## Naming

Apple naming conventions SHOULD be adhered to wherever possible, especially those related to [memory management rules](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html) ([NARC](http://stackoverflow.com/a/2865194/340508)).

Long, descriptive method and variable names are good.

**For example:**

```objc
UIButton *settingsButton;
```

**Not**

```objc
UIButton *setBut;
```

A three letter prefix (e.g., `NYT`) MUST be used for class names and constants, however MAY be omitted for Core Data entity names. Constants MUST be camel-case with all words capitalized and prefixed by the related class name for clarity. A two letter prefix (e.g., `NS`) is [reserved for use by Apple](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/DefiningClasses/DefiningClasses.html#//apple_ref/doc/uid/TP40011210-CH3-SW12).

**For example:**

```objc
static const NSTimeInterval NYTArticleViewControllerNavigationFadeAnimationDuration = 0.3;
```

**Not:**

```objc
static const NSTimeInterval fadetime = 1.7;
```

Properties and local variables MUST be camel-case with the leading word being lowercase.

Instance variables MUST be camel-case with the leading word being lowercase, and MUST be prefixed with an underscore. This is consistent with instance variables synthesized automatically by LLVM. **If LLVM can synthesize the variable automatically, then let it.**

**For example:**

```objc
@synthesize descriptiveVariableName = _descriptiveVariableName;
```

**Not:**

```objc
id varnm;
```

### Categories

* Categories are RECOMMENDED to concisely segment functionality and should be named to describe that functionality.

**For example:**

```objc
@interface UIViewController (NYTMediaPlaying)
@interface NSString (NSStringEncodingDetection)
```

**Not:**

```objc
@interface NYTAdvertisement (private)
@interface NSString (NYTAdditions)
```

* Methods and properties added in categories MUST be named with an app- or organization-specific prefix. This avoids unintentionally overriding an existing method, and it reduces the chance of two categories from different libraries adding a method of the same name. (The Objective-C runtime doesn’t specify which method will be called in the latter case, which can lead to unintended effects.)

**For example:**

```objc
@interface NSArray (NYTAccessors)
- (id)nyt_objectOrNilAtIndex:(NSUInteger)index;
@end
```

**Not:**

```objc
@interface NSArray (NYTAccessors)
- (id)objectOrNilAtIndex:(NSUInteger)index;
@end
```

* If you need to expose private methods for subclasses or unit testing, create a class extension named `Class+Private`.

## Comments

When they are needed, comments SHOULD be used to explain **why** a particular piece of code does something. Any comments that are used MUST be kept up-to-date or deleted.

Block comments are NOT RECOMMENDED, as code should be as self-documenting as possible, with only the need for intermittent, few-line explanations. This does not apply to those comments used to generate documentation.

## init and dealloc

`dealloc` methods SHOULD be placed at the top of the implementation, directly after the `@synthesize` and `@dynamic` statements. `init` methods SHOULD be placed directly below the `dealloc` methods of any class.

`init` methods should be structured like this:

```objc
- (instancetype)init 
{
    self = [super init]; // or call the designated initializer
    if (self) {
        // Custom initialization
    }

    return self;
}
```

## Literals

* `NSString`, `NSDictionary`, `NSArray`, and `NSNumber` literals SHOULD be used whenever creating immutable instances of those objects. Pay special care that `nil` values not be passed into `NSArray` and `NSDictionary` literals, as this will cause a crash.

**For example:**

```objc
NSArray *names = @[ @"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul" ];
NSDictionary *productManagers = @{ @"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill" };
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingZIPCode = @10018;
```

**Not:**

```objc
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingZIPCode = [NSNumber numberWithInteger:10018];
```

* Avoid making numbers a specific type unless necessary (for example, prefer `5` to `5.0`, and `5.3` to `5.3f`).
* The contents of array and dictionary literals should have a space on both sides.
* Dictionary literals should have no space between the key and the colon, and a single space between colon and value.

**For example:**
``` objc
NSArray *theStuff = @[ @1, @2, @3 ];

NSDictionary *keyedStuff = @{ GHDidCreateStyleGuide: @YES };
```

 * Longer or more complex literals should be split over multiple lines (optionally with a terminating comma):

**For example:**
``` objc
NSArray *theStuff = @[
    @"Got some long string objects in here.",
    [AndSomeModelObjects too],
    @"Moar strings."
];

NSDictionary *keyedStuff = @{
    @"this.key": @"corresponds to this value",
    @"otherKey": @"remoteData.payload",
    @"some": @"more",
    @"JSON": @"keys",
    @"and": @"stuff",
};
```

## `CGRect` Functions

When accessing the `x`, `y`, `width`, or `height` of a `CGRect`, code MUST use the [`CGGeometry` functions](http://developer.apple.com/library/ios/#documentation/graphicsimaging/reference/CGGeometry/Reference/reference.html) instead of direct struct member access. From Apple's `CGGeometry` reference:

> All functions described in this reference that take CGRect data structures as inputs implicitly standardize those rectangles before calculating their results. For this reason, your applications should avoid directly reading and writing the data stored in the CGRect data structure. Instead, use the functions described here to manipulate rectangles and to retrieve their characteristics.

**For example:**

```objc
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
```

**Not:**

```objc
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
```

## Constants

Constants are RECOMMENDED over in-line string literals or numbers, as they allow for easy reproduction of commonly used variables and can be quickly changed without the need for find and replace. Constants MUST be declared as `static` constants. Constants MAY be declared as `#define` when explicitly being used as a macro.

**For example:**

```objc
static NSString * const NYTAboutViewControllerCompanyName = @"The New York Times Company";

static const CGFloat NYTImageThumbnailHeight = 50.0;
```

**Not:**

```objc
#define CompanyName @"The New York Times Company"

#define thumbnailHeight 2
```

## Enumerated Types

When using `enum`s, the new fixed underlying type specification MUST be used; it provides stronger type checking and code completion. The SDK includes a macro to facilitate and encourage use of fixed underlying types: `NS_ENUM()`.

**Example:**

```objc
typedef NS_ENUM(NSInteger, NYTAdRequestState) {
    NYTAdRequestStateInactive,
    NYTAdRequestStateLoading
};
```

## Bitmasks

When working with bitmasks, the `NS_OPTIONS` macro MUST be used.

**Example:**

```objc
typedef NS_OPTIONS(NSUInteger, NYTAdCategory) {
    NYTAdCategoryAutos      = 1 << 0,
    NYTAdCategoryJobs       = 1 << 1,
    NYTAdCategoryRealState  = 1 << 2,
    NYTAdCategoryTechnology = 1 << 3
};
```

## Case Statements

Braces are not required for case statements, unless enforced by the complier (compiler enforces them when there are variable declaration inside).  

**For example:**
```objc
switch (condition) {
  case 1:
    // ...
    break;
  case 2: {
    // ...
    // variable declaration
    break;
  }
  case 3:
    // ...
    break;
  default: 
    // ...
    break;
}
```

There are times when the same code can be used for multiple cases, and a fall-through should be used.  A fall-through is the removal of the 'break' statement for a case thus allowing the flow of execution to pass to the next case value.  A fall-through should be commented for coding clarity.

**For example:**
```objc
switch (condition) {
  case 1:
    // ** fall-through! **
  case 2:
    // code executed for values 1 and 2
    break;
  default: 
    // ...
    break;
}
```

When using an enumerated type for a switch, 'default' is not needed.  

**For example:**
```objc
RWTLeftMenuTopItemType menuType = RWTLeftMenuTopItemMain;

switch (menuType) {
  case RWTLeftMenuTopItemMain:
    // ...
    break;
  case RWTLeftMenuTopItemShows:
    // ...
    break;
  case RWTLeftMenuTopItemSchedule:
    // ...
    break;
}
```

## Loop Statements

### For

When iterating using integers, it is preferred to start at `0` and use `<` rather than starting at `1` and using `<=`. Fast enumeration is generally preferred.

```objc
// enumeration using integer
for (NSInteger i = 0; i < 10; i++) {
    // Do something
}

// fast enumeration
for (NSString *key in dictionary) {
    // Do something
}
```

### While

```objc
while (something < somethingElse) {
    // Do something
}
```

## Private Properties

Private properties SHALL be declared in class extensions (anonymous categories) in the implementation file of a class.

**For example:**

```objc
@interface NYTAdvertisement ()

@property (nonatomic, strong) GADBannerView *googleAdView;
@property (nonatomic, strong) ADBannerView *iAdView;
@property (nonatomic, strong) UIWebView *adXWebView;

@end
```

## Image Naming

Image names should be named consistently to preserve organization and developer sanity. Images SHOULD be named as one camel case string with a description of their purpose, followed by the un-prefixed name of the class or property they are customizing (if there is one), followed by a further description of color and/or placement, and finally their state.

**For example:**

* `RefreshBarButtonItem` / `RefreshBarButtonItem@2x` and `RefreshBarButtonItemSelected` / `RefreshBarButtonItemSelected@2x`
* `ArticleNavigationBarWhite` / `ArticleNavigationBarWhite@2x` and `ArticleNavigationBarBlackSelected` / `ArticleNavigationBarBlackSelected@2x`.

Images that are used for a similar purpose SHOULD be grouped in respective groups in an Images folder or Asset Catalog.

## Booleans

Values MUST NOT be compared directly to `YES`, because `YES` is defined as `1`, and a `BOOL` in Objective-C is a `CHAR` type that is 8 bits long (so a value of `11111110` will return `NO` if compared to `YES`).

**For an object pointer:**

```objc
if (!someObject) {
}

if (someObject == nil) {
}
```

**For a `BOOL` value:**

```objc
if (isAwesome)
if (!someNumber.boolValue)
if (someNumber.boolValue == NO)
```

**Not:**

```objc
if (isAwesome == YES) // Never do this.
```

If the name of a `BOOL` property is expressed as an adjective, the property’s name MAY omit the `is` prefix but should specify the conventional name for the getter.

**For example:**

```objc
@property (assign, getter=isEditable) BOOL editable;
```

_Text and example taken from the [Cocoa Naming Guidelines](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE)._

## Singletons

Singleton objects SHOULD use a thread-safe pattern for creating their shared instance.
```objc
+ (instancetype)sharedInstance 
{
    static id sharedInstance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[[self class] alloc] init];
    });

    return sharedInstance;
}
```
This will prevent [possible and sometimes frequent crashes](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html).

## Imports

* `#import` statements SHOULD NOT be written in header files, and SHOULD be used only in implementation files. Use forward class declarations instead. Forward class declarations in lieu of #importing headers will lead to faster compile times, will avoid circular #import statements, and will keep your headers lightweight, the way they were meant to be.
The one real exception is when subclassing another custom class, you’ll need to #import its header. 

**For example:**
```objc
// MyClass.h

@class MyOtherClass;

@interface MyClass : NSObject

@property (nonatomic, strong) MyOtherClass property;

@end

// MyClass.m
#import "MyOtherClass.h"
```

* If there is more than one import statement, statements MUST be grouped [together](http://ashfurrow.com/blog/structuring-modern-objective-c). Groups MAY be commented.

Note: For modules use the [@import](http://clang.llvm.org/docs/Modules.html#using-modules) syntax.

**For example:**
```objc
// Frameworks
@import QuartzCore;

// Models
#import "NYTUser.h"

// Views
#import "NYTButton.h"
#import "NYTUserView.h"
```

* Precompiled header (.pch) file SHOULD be used to avoid standard imports. All imports written there will be added to all your headers automatically by XCode

**For example:**
```objc
// AppName-Prefix.pch
#import <UIKit/UIKit.h>
#import <Foundation/Foundation.h>
```

## Protocols

In a [delegate or data source protocol](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html), the first parameter to each method SHOULD be the object sending the message.

This helps disambiguate in cases when an object is the delegate for multiple similarly-typed objects, and it helps clarify intent to readers of a class implementing these delegate methods.

**For example:**

```objc
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;
```

**Not:**

```objc
- (void)didSelectTableRowAtIndexPath:(NSIndexPath *)indexPath;
```

## Golden Path

When coding with conditionals, the left hand margin of the code should be the "golden" or "happy" path.  That is, don't nest `if` statements.  Multiple return statements are OK.

**For example:**

```objc
- (void)someMethod {
  if (![someOther boolValue]) {
    return;
  }

  //Do something important
}
```

**Not:**

```objc
- (void)someMethod {
  if ([someOther boolValue]) {
    //Do something important
  }
}
```

## Xcode project

The physical files SHOULD be kept in sync with the Xcode project files in order to avoid file sprawl. Any Xcode groups created SHOULD be reflected by folders in the filesystem. Code SHOULD be grouped not only by type, but also by feature for greater clarity.

Target Build Setting “Treat Warnings as Errors” SHOULD be enabled. Enable as many [additional warnings](http://boredzo.org/blog/archives/2009-11-07/warnings) as possible. If you need to ignore a specific warning, use [Clang’s pragma feature](http://clang.llvm.org/docs/UsersManual.html#controlling-diagnostics-via-pragmas).

## Commit messages

A properly formed Git commit subject line MUST be able to complete the following sentence:

(If applied, this commit will) \<your subject line here\>

**For example:**
```
(If applied, this commit will) refactor subsystem X for readability
(If applied, this commit will) update getting started documentation
(If applied, this commit will) remove deprecated methods
(If applied, this commit will) release version 1.0.0
(If applied, this commit will) merge pull request #123 from user/branch
```

Notice how this doesn't work for the other non-imperative forms:

**Not:**
```
(If applied, this commit will) fixed bug with Y
(If applied, this commit will) changing behavior of X
(If applied, this commit will) more fixes for broken stuff
(If applied, this commit will) sweet new API methods
```

More on writing commit messages: http://chris.beams.io/posts/git-commit/


## Boyscout
Whenever you are in a piece of code it should be left cleaner than when you found it if possible. If you find code that violates this guide, correct it. If the code is out dated then update it.

# Other Objective-C Style Guides

If ours doesn’t fit your tastes, have a look at some other style guides:

* [Google](https://google.github.io/styleguide/objcguide.xml)
* [GitHub](https://github.com/github/objective-c-style-guide)
* [Adium](https://trac.adium.im/wiki/CodingStyle)
* [Sam Soffes](https://gist.github.com/soffes/812796)
* [CocoaDevCentral](http://cocoadevcentral.com/articles/000082.php)
* [Luke Redpath](http://lukeredpath.co.uk/blog/2011/06/28/my-objective-c-style-guide/)
* [Marcus Zarra](http://www.cimgf.com/zds-code-style-guide/)
* [Wikimedia](https://www.mediawiki.org/wiki/Wikimedia_Apps/Team/iOS/ObjectiveCStyleGuide)
