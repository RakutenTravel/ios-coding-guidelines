# iOS Coding Guidelines

This page tries to explain the coding guidelines we follow not only in terms of code style, but also in regards to general coding practices. This document will be updated as Swift continues to evolve, but please make sure to give this a quick read so we can keep our code standardized.

## Table of Contents

- [1. Base Coding Style](#1-base-coding-style)  
    * [1.1 Swift API Design Guidelines](#11-swift-api-design-guidelines) 
    * [1.2 SwiftLint](#12-swiftlint) 
- [2. DO. NOT. HARDCODE](#2-do-not-hardcode)  
- [3. Naming Conventions](#3-naming-conventions)  
    * [3.1 Property names with ambiguous type information](#31-property-names-with-ambiguous-type-information) 
- [4. \[Alt\]SwiftUI](#4-altswiftui) 
    * [4.1 Views](#41-views)
    * [4.2 Modifiers](#42-modifiers)
    * [4.3 Opaque Types](#43-opaque-types)
- [5. Code Structure](#5-code-structure) 
    * [5.1 File Type Structures](#51-file-types-structure)
    * [5.2 Main Type Structures](#52-main-type-structure)
    * [5.3 View's Wrapped Properties Structure](#53-views-wrapped-properties-structure)
    * [5.4 Mutability and Accessibility-wise](#54-mutability-and-accessibility-wise)
    * [5.5 Overall Example](#55-overall-example)
- [6. Line Wrapping](#6-line-wrapping)
    * [6.1 Line Width](#61-line-width)
    * [6.2 Where To Break Lines](#62-where-to-break-lines)
    * [6.3 Multiple Parameters & Trailing Closures](#63-multiple-parameters--trailing-closures)
    * [6.4 Chained Function Calls](#64-chained-function-calls)
- [7. About MARKs](#7-about-marks) 
- [8. Extensions Usage](#8-extensions-usage) 
    * [8.1 Implementation of a Protocol](#81-implementation-of-a-protocol) 
    * [8.2 Declaration of Related Fileprivate Properties from Other Types](#82-declaration-of-related-fileprivate-properties-from-other-types) 
    * [8.3 Structs' Custom Inits](#83-structs-custom-inits) 
- [9. Control Transfer Statements](#9-control-transfer-statements) 
    * [9.1 `Guard` vs `If`](#91-guard-vs-if) 

## 1. Base Coding Style

### 1.1 Swift API Design Guidelines

Please, before reading all of this, be sure that you know all the basic official [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/).

### 1.2 SwiftLint

**swift-lint** helps us enforce many small coding rules. Please be sure to have it installed and running on your machine. The rules used are specified on the **\`.swiftlint.yml\`** file (it is hidden by default)

The library has also their own [coding style guide](https://github.com/github/swift-style-guide) but for more information about the rules we have chosen, please refer [here](https://realm.github.io/SwiftLint/rule-directory.html).

## 2. DO. NOT. HARDCODE

Just don't. :neutral_face: Please. :relieved:

## 3. Naming Conventions

### 3.1 Property names with ambiguous type information

Some property names might be misleading. Especially if you can't rely on the IDE at the moment (e.g: While reviewing a PR, or when XCode's indexing fails)

Imagine the following code to review in your PR:

```swift
Image(image: providerImage)
```

You may think this is receiving an UIImage. However when you check the actual code, which is not included in the PR, you find this.

```swift
private var providerImage: String? {
    provider.media?.first?.thumbnail ?? provider.media?.first?.url
}
```

In cases like this, try compensating for the weak type information on your property's name.

For the case above, the better name would be \`**providerImageURLString**\`

## 4. [\[Alt\]](https://github.com/rakutentech/AltSwiftUI)SwiftUI

### 4.1 Views

<details>
  <summary>Maximum Nesting Levels</summary>
  
Maximum **3 levels** of nesting, without counting **Stacks** (VStack, HStack, ZStack), **Lists** (List, ForEach) or **ScrollViews**.

```swift
VStack {
  Text("Title") // 1st level
  HStack {
      Text("Subtitle") // 2nd level
      VStack {
          Text("Price") // 3rd level
          Text("USD 10.00")
      }
  }
}
``` 

We also strongly suggest refactoring the following components into separate computed properties/functions/files:

*   Cells
*   Headers
*   Footers
*   Any other view that requires a closure to define its own subcomponents (e.g. PageViewController)
</details>

<details>
  <summary>Button</summary>
    
If you want to add custom views to a button, remember to put **everything** inside, so that all the content is clickable.

_e.g. of wrong setup_: `Button(){}.padding().cornerRadius()` -> Will create un-clickable content.

- accent
    * If you want to change the accent of a button, always apply the accent in the button itself, instead of inside its content.

- foregroundColor
    * You can specify foreground color either at Button level or content level, depending if you want to target specific content only.
</details>

<details>
  <summary>ForEach</summary>
    `ForEach` represents a loop. Don't add dimension properties like frame and padding to this view!
</details>

### 4.2 Modifiers

<details>
  <summary>.animation()</summary>
    
*   (RakuSwiftUI limitation) Make sure any properties you wish to animate are set **after** any padding() setting.
*   Use .animation() selectively as it affects everything that's defined before it. For transitions, apply the animation directly to the transition, or in general, use withAnimation()
</details>

<details>
  <summary>.environmentObject()</summary>
    
Don't set the same environment object type twice. Make sure that only one environment object of the same type is needed at a time. Redeclaring environment objects in sub-flows can make the data flow confusing and prone to errors.

Better solution: Use _Binding_ or _ObservedObject_.
</details>

<details>
  <summary>.frame() and .padding()</summary>

These functions create **new views.** Any modifiers applied after this are applied to the new view, and not the original one.  

e.g. Image().frame().scaledToFill() -> This will not scale to fill the image, but the frame, causing undesired results!  
</details>

<details>
  <summary>.onAppear()</summary>

Only use onAppear on the top-most view of a body or cell, or on the top-most view after an _if_ / _if-else_ / _ForEach_ statement.
</details>

<details>
  <summary>.padding()</summary>

If you want to add 2 types of padding, use .padding(EdgeInsets) **instead** of using 2 or more .padding() functions
</details>

### 4.3 Opaque Types

In contrast to the actual SwiftUI, RakuSwiftUI does not support [opaque types](https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html). However, because we are planning to eventually migrate the code to SwiftUI, we would like to keep code as similar as possible to avoid compiling errors in the future.

Hence, whenever we use a getter that has an opaque type as its specified return value (e.g: \`**View**\`), we should only return 1 single concrete class as the return type. This is because once we migrate to SwiftUI these methods will return \`**some View**\` instead of \`**View**\`, and if we don't respect this rule the compiler will tell you to do this.

<details>
  <summary>:x: BAD</summary>
  
  ```swift
  // This is returning 2 concrete classes, `EmptyView` and `VStack`. Will fail in SwiftUI.

  var mySubview: View {
      guard myBoolean else { return EmptyView() } 
      return VStack {
          ...
      }
  }
  ```
</details>

<details>
  <summary>:white_check_mark: GOOD</summary>
  
  ```swift
  // Returning VStack in both cases. Won't fail in SwiftUI and the result is the same.

  var mySubview: View {
      guard myBoolean else { return EmptyView() } 
      return VStack {
          if myBoolean {
              ...
          }
      }
  }
  ```
</details>

### 5. Code Structure

Our files code is going to be grouped and ordered, using the following prioritization.

### 5.1 File Types Structure

The 1st priority states the following grouping and order.

1.  Supporting types (Protocols, Enums)
2.  "Custom init"-related Struct Extensions
3.  Main Type (i.e View)
4.  Extensions

### 5.2 Main Type Structure

The 2nd priority states the overall structure of the main type's contents (i.e. your View, Model, etc). This should be separated by `MARK`s (without horizontal rule)

1.  Inner types (inner Enum, Struct, Tuples, typealiases)
2.  Properties included in the **init**.
3.  viewStore (for Views only)
4.  Properties with property-wrappers (For views, further ordering is defined on 5.3)
5.  Other non-subview related properties (they go after the body)
6.  Body (if View)
7.  Subviews logic (private functions and/or computed properties used to compute subviews internal information like texts or even entire views)
8.  Other functions

### 5.3 View's Wrapped Properties Structure

The 3rd priority is related to Views specifically and it states the structure of properties that make use of property wrappers.

1.  Environment
2.  EnvironmentObject
3.  State
4.  ObservedObject
5.  Binding

### 5.4 Mutability and accessibility-wise

The 4th priority states the structure in terms of just the mutability and accessibility of the properties

1.  static lets
2.  static vars
3.  lets
4.  vars
5.  static computed properties
6.  computed properties
7.  functions
    
### 5.5 Overall Example

<details>
  <summary>Example</summary>
  
  ```swift
  enum MyEnum: String { ... }
  protocol MyViewDelegate { ... }

  extension MyView {
      init(customInitParameter: ...)
  }

  struct MyView: View {
    // MARK: Aliases

    typealias AwesomeInfo = (Int, Int)
      // MARK: Init Properties

      var title: String    
      var media: [Media]
      @Binding var isSheetPresented: Bool


      static var `default`: MyView {
      MyView(title: "", media: [], isSheetPresented: false)
    }

      // MARK: State-related properties

      var viewStore = ViewValues()    

      @Environment (\\.presentationMode) var presentationMode
      @EnvironmentObject var searchQuery: SearchQueryModel
      @State private var showSearchResults = false
      @State private var showMoreAreas = false
      @ObservedObject var myViewModel: MyViewModel
      var isFirstLoad: Binding<Bool>?

      static let fadeAnimationDelay = 3.5
      let sidePadding: CGFloat = CGFloat.sidePadding + 3.0

      // MARK: Body

      var body: View {
          ...
      }

      // MARK: Subviews Logic

      private var contentView: View { 
          ... 
      }

      private var keywords: String { searchQuery.keywords.displayText }

      // MARK: Other functions

      private func collapseAllCells() {
          ...
      } 
  }

  extension MyView: UITableViewDelegate { ... }
  ```
</details>

## 6. Line Wrapping

### 6.1 Line Width

For a quick criteria, try wrapping whenever you surpass **column 130** on your line.

### 6.2 Where To Break Lines

For more details on **where** in your line you should break it, please follow [Google's Swift Style Guide](https://google.github.io/swift/#line-wrapping).

Especially, try wrapping when you have initializers or functions with a big amount of parameters.

<details>
  <summary>:x: BAD</summary>
  
  ```swift
  BookingStep1View(bookingSelection: BookingSelection(providerId: self.provider.id,
                                                      roomId: self.room.id,
                                                      planId: self.plan.planId,
                                                      itemPlanId: self.plan.id,
                                                      room: self.room,
                                                      provider: self.provider),
                   providerDetailModel: self.providerDetailModel,
                   showBookingStep: self.$showBooking,
                   parentViewLoadingViewPresented: self.$blockingLoadingViewPresented,
                   planUnavailableGuideSelection: self.sharedPlanUnavailableGuideSelection,
                   imageCache: self.imageCache)
  ```
</details>

<details>
  <summary>:white_check_mark: GOOD</summary>
  
  ```swift
  BookingStep1View(
      bookingSelection: BookingSelection(
          providerId: self.provider.id,
          roomId: self.room.id,
          planId: self.plan.planId,
          itemPlanId: self.plan.id,
          room: self.room,
          provider: self.provider),
      providerDetailModel: self.providerDetailModel,
      showBookingStep: self.$showBooking,
      parentViewLoadingViewPresented: self.$blockingLoadingViewPresented,
      planUnavailableGuideSelection: self.sharedPlanUnavailableGuideSelection,
      imageCache: self.imageCache)
  ```
</details>

### 6.3 Multiple Parameters & Trailing Closures

As inits and functions cannot only be line-wrapped throughout multiple lines, but also have trailing closures, we are defining the following 3 patterns:

#### 6.3.1 Single-line Pattern

The simplest pattern. No need to break lines if everything fits into one single line.

<details>
  <summary>Example</summary>
  
  ```swift
  // 1. 
  MyClass(foo: foo, bar: bar)

  // 2.
  MyClass(foo: foo) { print($0) }

  // 3.
  self.myMethod(foo: foo) { print($0) }
  ```
</details>


#### 6.3.2 Multiline Parameters Without Trailing Closure

Whenever you are breaking your parameters into different lines **and no trailing closure is declared**, the closing parentheses MUST be in the same line as the end of the last parameter.

<details>
  <summary>:white_check_mark: GOOD</summary>
  
  ```swift
  // 1.
  MyClass(
      foo: foo,
      bar: bar)

  // 2.
  MyClass(
      foo: foo,
      bar: bar,
      where: { 
          $0 == true
      },
      includedIn: { 
          $0 == true
      })

  // 3.
  self.request(
      foo: foo,
      bar: bar,
      onSuccess: { 
          print($0)
      },
      onFailure: { 
          print($0)
      })
  ```
</details>

#### 6.3.2 Multiline Parameters With Trailing Closure

Whenever you are breaking your parameters into different lines **and a trailing closure is declared**, the init's closing parentheses and the trailing closure's opening bracket should be on a new line.

<details>
  <summary>Example</summary>
  
  ```swift
  // 1.
  MyClass(
      foo: foo,
      bar: bar
  ) {
      print($0)
  }
  ```
</details>

### **6.4 Chained Function Calls**

When breaking lines while using functions calls that could be chained (i.e builder functions, modifiers, high-order functions), the indentation of each call will depend on the first instruction of the chain, based on the following patterns.

#### 6.4.1 Single-lined First Instruction

If the first line of the chain fits in only one line, the following function calls will all have only **one more level** of indentation than the first line.

<details>
  <summary>Example</summary>
  
  ```swift
  // 1.
  self.request(foo: foo)
      .success({
          ...
      })
      .failure( {
          ...
      })
      .send()

  // 2. 
  MyClass(foo: foo, bar: bar) { print($0) }
      .modified(by: something)
      .modifiedAgain(by: somethingElse)

  // 3.
  [1, 2, 3]
      .map { $0 + 1 }
      .filter { $0 < 2 }
  ```
</details>

#### 6.4.2 Multi-lined First Instruction

If the first instruction of the chain goes across multiple lines, the following function calls will all have **the same level** of indentation as the line including the closing element (either parentheses **\`)\`** or curly brace **\`}\`**).

<details>
  <summary>Example</summary>
  
```swift

  // 1.

  MyClass(
      foo: foo, 
      bar: bar
  ) { 
      print($0) 
  }
  .modified(by: something)
  .modifiedAgain(by: somethingElse)

  // 2.
  MyView(
      foo: foo,
      bar: bar)
      .success({
          ...
      })
      .failure( {
          ...
      })
      .send()
  ```
</details>

## 7. About MARKs

*   MARKs should have **horizontal rules (\`// MARK: -\`)** only if they are at the file level.
    *   i.e. before the declaration of a group of extensions, not inside them
*   MARKs need to have a **white line** before and after their declaration.
*   MARKs should only to **group up/separate** related logic will be declared onwards.
    *   e.g: Private extensions, Private methods
*   When naming your MARK, try giving more specific information about the grouping.
    *   i.e Instead of just grouping something as \`Private methods\`, maybe you can call it \`Subviews computation\`, \`Size calculation\`, \`API calls\`, etc.  
          
<details>
  <summary>Example</summary>
  
  ```swift
  // 1. There is only one struct in the entire file. No need to add a MARK here as we are not grouping anything, not organizing anything.

  // MARK: - MyRequest
  struct MyRequest {
  }

  // 2. Bad use of MARKs rules. Lack of white lines

  // MARK: MyRequest
  struct MyRequest {
    // MARK: - init
      init(...) {
      }
  }

  struct MyView: View {
      var otherView: MyOtherView
      var anotherView: AnotherView

      public var body: View {
          Text(self.otherView.formattedText)
          Text(self.anotherView.elementsCountText)
      }

      // MARK: Private methods

      func myPrivateMethod() { ... }
  }

  // MARK: - Subview computation

  extension MyOtherView {
      fileprivate var formattedText: String { ... }
  }

  extension AnotherView {
      fileprivate var elementsCountText: String { ... }
  }
  ```
</details>

## 8. Extensions Usage

Use extensions for the following cases.

### 8.1 Implementation of a Protocol

Use extensions to separate the declaration of a Type from the implementation of a protocol by that same Type.

<details>
  <summary>Example</summary>
  
```swift
struct CustomView { ... }

extension CustomView: UICollectionViewDelegate { ... }
```
</details>

### 8.2 Declaration of Related Fileprivate Properties from Other Types

Use **fileprivate** extensions whenever you need to extend other Types functionality, in order to support your the Type declared in your file. Prioritize using **computed properties** over creating \`**getter**\` methods like **\`getTitle(from: otherObject)\`**, **\`calculate(from: array)\`.**

<details>
  <summary>Example</summary>
  
  ```swift
  struct CustomView {
      var body: View { 
          Text(self.getFormattedTitle(from: self.provider))
      }

      // GETTER METHOD DEPENDING ON OTHER TYPE.
    fileprivate var getFormattedTitle(from provider: Provider) -> String {
      ...
      }   
  }

  struct CustomView {
      var body: View { 
          Text(self.provider.formattedTitle)
      }
  }

  extension Provider {
      // CLEANER APPROACH
      fileprivate var formattedTitle: String {
      ...
      }     
  }
  ```
</details>

### 8.3 Structs' Custom Inits

In the case of **structs**, in order to avoid deleting the default **member-wise initializer** that is provided by default, make use of extensions to define your custom inits.

However, this extension should be directly **above** the struct's definition, for better visibility.

<details>
  <summary>Example</summary>
  
  ```swift
  struct Position {
      var x: Int
      var y: Int

      // SAME AS MEMBERWISE INIT!!
      init(x: Int, y: Int) {
          self.x = x
          self.y = y
      }

      init(point: CGPoint) {
          self.x = point.x
          self.y = point.y
      }
  }

  extension Position {
      init(point: CGPoint) {
          self.x = point.x
          self.y = point.y
      }
  }

  struct Position {
      var x: Int
      var y: Int
  }
  ```
</details>

## 9. Control Transfer Statements

### 9.1 \`Guard\` vs \`If\`

Although similar in the sense that both allows us to handle control flow based on boolean conditions, `guard` has a nuance of **verifying something before continuing with the current flow** (hence the name \`guard\`). So, if your algorithm requires some verification, or maybe it needs to unwrap certain things in order for it to advance with the rest of the code, then you should prioritize guard.

Another good point of `guard` is that it allows you to avoid unnecessary nesting that would happen if you use `if`.

<details>
  <summary>Example</summary>

  ```swift
  func myFunc() -> View {
      if myList.isEmpty {
          return VStack { }        
      } else {
          return VStack {
              ... lots of code ... // Due to the `if` above, now the rest of the code is indented unnecesarily.
          }
      }
  }

  func myFunc() -> View {
      // No point in continuing with the algorithm if there are no items in my list. Hence, `guard` is better here.
      guard !myList.isEmpty else { return VStack { } } // PROTIP: You can also use one-lined guards if they are simple ;)

      return VStack {
          ... lots of code ... // One level of indentation avoided!
      }
  }
  ```
</details>

On the other hand, if you find that the body of the `else` in your guard statement has complex logic or even nested control flow inside, then is more than likely that you would be better with an `if` statement.

Also, if you find that because of `guard` your conditions seem a bit difficult to read (because you need to use the **negative** version of your them), you can opt to refactor your booleans before the guard statement (break them down into multiple booleans with more understandable names).

(Maybe your condition was difficult to read to begin with and you just realize it because of the guard...?)

<details>
  <summary>:x: BAD</summary>
  
  ```swift
  // 1. Complex guard conditions.

  func myFunc() -> Void {
      // The condition of the guard might be difficult to read.
      guard !((condition1 && condition2) || (condition3 && condition4)) else { return }

      ... lots of code ... 
  }


  // 2. Complex guard body

  func myFunc() -> Void {
      // This guard does not provide a \*fast return\*, but rather it goes to another flow with its own logic.
      guard myBoolean else { 
          if otherBoolean {
              ...
          } else {
              ...
          }
      }

      ... lots of code ... 
  }
  ```
</details>

<details>
  <summary>:white_check_mark: GOOD</summary>
  
  ```swift
  // 1. Complex guard conditions.


  func myFunc() -> Void {
      // The condition of the guard might be difficult to read.
      let compoundCondition1 = (condition1 && condition2)
      let compoundCondition2 = (condition3 && condition4)
      let complexCondition = compoundCondition1 || compoundCondition2

      guard !(complexCondition) else { return }

      ... lots of code ... 
  }

  // 2. Complex guard body

  func myFunc() -> Void {
      // If the guard body is complex, we use instead an if.
      if !myBoolean { 
          if otherBoolean {
              ...
          } else {
              ...
          }
      } else {
          ... lots of code ... 
      }
  }
  ```
</details>
