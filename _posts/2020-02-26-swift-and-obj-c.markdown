---
layout: post
title: "Mixing Swift and Objective‑C"
date: 2020-02-26
categories: etc
---

I was responsible for migrating a very large Objective‑C app to Swift. To limit
effort, we decided not to rewrite the entire thing in Swift (that would have
taken years, probably). Instead we decided to do all new development in Swift,
and to simply add Swift compatibility as-needed to the codebase.

The official docs are okay on this topic:

* [Using Objective‑C in Swift][objcinswift]
* [Using Swift in Objective‑C][swiftinobjc]

[objcinswift]: https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift
[swiftinobjc]: https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_swift_into_objective-c

But I developed this guide to more quickly help other developers of the app with
the tricky situations we often ran into.

## Making Objective‑C nicer to use in Swift

* Annotate types in your headers with `nullable` or `nonnull` so they do not get
  imported as implicitly unwrapped optionals ([more info][nullable])
* Declare enums using `NS_ENUM` and options sets using `NS_OPTIONS` ([more
  info][enums])
* Declare custom NSError codes using `NS_ERROR_ENUM` ([more info][error])
* Declare methods that can return an NSError by making their last parameter a
  `NSError**` named `error`, such that Swift converts it to a `throws` method
* Swift should be able to import Objective‑C-style method
  names `doThingWithArg1:AndArg2:` as Swift-style method names
  `doThing(arg1:arg2:)`. If Swift imports something with an Objective‑C-style
  method name, use `NS_SWIFT_NAME` to fix it ([more info][names])

[nullable]: https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/designating_nullability_in_objective-c_apis
[enums]: https://nshipster.com/ns_enum-ns_options/
[error]: https://developer.apple.com/documentation/swift/cocoa_design_patterns/handling_cocoa_errors_in_swift
[names]: https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/renaming_objective-c_apis_for_swift

## Swift class name gotchas

By default, Swift class names are prefixed by a namespace and Objective‑C
classes are not.

For example, if you add a class to DataLayerFramework in Swift its full name is
DataLayerFramework.NewClass but in Objective‑C it's just NewClass. Examples of
where this matters and will cause errors:

* You're trying to get the class name as a string in Objective‑C. You need to
  use `NSStringFromClass(myClass).pathExtension` to drop the namespace
* You're trying to get the class from a string in Objective‑C. You either need
  the namespace `NSClassFromString(@"NameSpace.MyClass")` or rename the Swift
  class in Objective‑C by marking it with `@objc(MyClass) public class MyClass`
* When using a Swift class for an entity in CoreData, the namespace matters.
  Select the entity in the data model and check the right pane where it says
  "Module". This determines what namespace is used for looking up the class. If
  you use the wrong one, CoreData will fail to construct entities of that type
  (you'll get "unrecognized selector" errors because the entities won't be the
  right class).
  * For example, if you rename the Swift class using `@objc(MyClass)` you should
    clear the module field so that it says "Global namespace"
  * You can open the data model XML in a text editor to see what the actual
    lookup will be. It's the value of the `representedClassName` attribute

## Trying to use Swift stuff in an Objective‑C .m file

First make sure you aren't trying to use something that is Swift-only and thus
not visible in Objective‑C:

* Generics
* Tuples
* enums without Int raw value type
* structs
* Top-level functions (outside a class)
* Global variables (outside a class)
* Typealiases
* Swift-style variadic functions
* Nested types
* Curried functions
* Inheriting from Swift classes

If the thing is not on that list, then:

1. If the Swift is in a framework, all Swift
   classes/enums/inits/methods/properties you need to use must be marked public
   and @objc
   * You can omit the @objc if Swift is inheriting/overriding something from
     Objective‑C, but it never hurts to include it
2. Import in the .m depending on where the Swift is located
   * If in a framework, `#import
     <FrameworkModuleName/FrameworkModuleName-Swift.h>`
   * If in an app, `#import "Objective‑C_Generated_Interface_Header_Name.h"`
     (you can find this in Build Settings for the app target e.g.
     `"MyApp-Swift.h"`)

#### Error messages

* "Expected a type"
* "Cannot load nib in bundle xxxx"

## Trying to use Swift stuff in an Objective‑C .h file

See the note in the previous section about Swift stuff that cannot be used in
Objective‑C, then:

* If the Swift is in a different framework from the .h, `#import
  <FrameworkModuleName/FrameworkModuleName-Swift.h>`
* Else if they're in the same target, do not import. Instead forward-declare:
  * For a class, `@class MyClass;` or protocol `@protocol MyProtocol;`
  * For an enum, `typedef NS_ENUM(NSInteger, MyEnum);` (make sure the type
    matches the raw enum type)
* Note that Objective‑C classes cannot inherit from Swift classes

#### Error messages:

* "Expected a type"
* "Cannot find interface declaration"

## Trying to use Objective‑C stuff in Swift, the Objective‑C is in a framework (not in an app)

* Import the Objective‑C header in the umbrella header for the framework as
  `#import <ModuleName/HeaderName.h>`
  * If there is no umbrella header add one. At the root of the framework add a
    public header named `ModuleName.h`
  * You might need to import Foundation or UIKit in the umbrella header if your
    framework depends on those things
* Ensure the header has public visibility in the framework
* Apply 1 and 2 recursively: if the header imports other headers from the
  framework they should also be public and in the umbrella header, and so on
* If your Swift file is not in that framework, `import ModuleName`

## Trying to use Objective‑C stuff in Swift, not in a framework (e.g. just in an app)

* Import the header into the Swift bridging header
* If there is no bridging header, you can check the filename for it in the
  target's build settings. You can set it manually and create that file if none
  exists for some reason
* If Swift says it can't find something that you put in the bridging header, try
  building first to regenerate the header. Usually the error goes away.
  * It won't regenerate the header if something else is stopping it from
    building, like errors in framework dependencies. Xcode > Preferences >
    General > "Continue building after errors" can help here

## Swift header not found

The -Swift header is only created if Swift files are in the framework

* Check that the Swift files are in the right location
* Make sure all Swift files have the right target membership

#### Error messages:

* "'FrameworkModuleName/FrameworkModuleName-Swift.h' file not found"
* "Header 'FrameworkModuleName-Swift.h' not found"
