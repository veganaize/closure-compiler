# Overview

> (copied from https://developers.google.com/closure/compiler/docs/js-for-compiler, included in the GitHub wiki to permit community editing)

The Closure Compiler can use data type information about JavaScript variables to provide enhanced optimization and warnings. JavaScript, however, has no way to declare types.

Because JavaScript has no syntax for declaring the type of a variable, you must use comments in the code to specify the data type.

The Closure Compiler's type language derives from the annotations used by the JSDoc document-generation tool, although over the years it has diverged. This document describes the set of annotations and type expressions that the Closure Compiler understands.

This table includes only tags that affect on the behavior of the Closure Compiler. For information about other JSDoc tags see the JSDoc Toolkit documentation.

## Table of Contents

  * [Type annotations](#type-annotations)
    + [`@constructor`](#constructor)
    + [`@enum {Type}`](#enum-type)
    + [`@extends {Type}`](#extends-type)
    + [`@implements {Type}`](#implements-type)
    + [`@interface` `@record`](#interface-record)
    + [`@lends {objectName}`](#lends-objectname)
    + [`@param {Type} varname` _Description_](#param-type-varname-description)
    + [`@return {Type}` _Description_](#return-type-description)
    + [`@template T`](#template-t)
    + [`@this {Type}`](#this-type)
    + [`@type {Type}`](#type-type)
      - [Type casting](#type-casting)
    + [`@typedef {Type}`](#typedef-type)
  * [Visibility checks](#visibility-checks)
    + [`@deprecated` _Description_](#deprecated-description)
    + [`@final`](#final)
    + [`@package`](#package)
    + [`@public`](#public)
    + [`@private`](#private)
    + [`@protected`](#protected)
  * [Other typechecker annotations](#other-typechecker-annotations)
    + [`@const`, `@const {type}`](#const-const-type)
    + [`@dict`](#dict)
    + [`@implicitCast`](#implicitcast)
    + [`@inheritDoc`](#inheritdoc)
    + [`@override`](#override)
    + [`@polymer`](#polymer)
    + [`@polymerBehavior`](#polymerbehavior)
    + [`@struct`](#struct)
    + [`@suppress {warningGroup1,warningGroup2}`](#suppress-warninggroup1warninggroup2)
    + [`@unrestricted`](#unrestricted)
  * [Other annotations](#other-annotations)
    + [`@define {Type}` _description_](#define-type-description)
    + [`@export`, `@export {SomeType}`](#export-export-sometype)
    + [`@externs`](#externs)
    + [`@fileoverview Description`](#fileoverview-description)
    + [`@license`, `@preserve` _Description_](#license-preserve-description)
    + [`@noalias`](#noalias)
    + [`@nocompile`](#nocompile)
    + [`@nocollapse`](#nocollapse)
    + [`@noinline`](#noinline)
    + [`@nosideeffects` `@modifies {this|arguments}`](#nosideeffects-modifies-thisarguments)
    + [`@throws {Type}`](#throws-type)

---

## Type annotations

Use these tags to inform the compiler of the type of a variable or to define a type you can use later. If you're new to the Closure Compiler, https://github.com/google/closure-compiler/wiki/Annotating-Types has some examples of how to annotate different kinds of declarations.

See https://github.com/google/closure-compiler/wiki/Types-in-the-Closure-Type-System for information on how the actual Closure type system works.

---

### `@constructor`

Marks a function as a constructor.  The compiler requires a `@constructor` annotation for any function that is used with the `new` keyword.  @constructor should be omitted from EcmaScript `class` constructor methods and `goog.defineClass` constructor methods.

```javascript
/**
 * A rectangle.
 * @constructor
 */
function GM_Rect() {
  ...
}
```

---

### `@enum {Type}`

Specifies an enum, which is a type with a specific finite number of possible values, often strings or numbers. `@enum` tag must be followed by a type expression.  If the type of an enum is omitted, number is assumed.

The type label of an enum applies to each property of the enum. For example if an enum has type number, each of its enumerated properties must be a number.

```javascript
/**
 * Enum for tri-state values.
 * @enum {number}
 */
project.TriState = {
  TRUE: 1,
  FALSE: -1,
  MAYBE: 0
};
```

---

### `@extends {Type}`

Marks a class or interface as inheriting from another class. A class marked with `@extends` must also be marked with either `@constructor`, `@interface`, or `@record`.

> Note: `@extends` does not cause a class to inherit from another class. The annotation simply tells the compiler that it can treat one class as a subclass of another during type-checking.

For an example implementation of inheritance, see the Closure Library function `goog.inherits()`.

```javascript
/**
 * Immutable empty node list.
 * @constructor
 * @extends {goog.ds.BasicNodeList}
 */
goog.ds.EmptyNodeList = function() {
  ...
};
```

While only one `@extends` is allowed with @constructor, it is allowed to use more than one for `@interface` or `@record`.

```javascript
/**
 * @interface
 * @extends {SuperInterface1}
 * @extends {SuperInterface2}
 */
function SomeInterface() {
  ...
};
```

> Note: `@extends` is implicit with `class` expressions but can be used explicitly to, for example, provide type arguments to the super class type.

---

### `@implements {Type}`

Used with `@constructor` to indicate that a class implements an interface.

The compiler produces a warning if you tag a constructor with `@implements` and then fail to implement all of the methods and properties defined by the interface.

```javascript
/**
 * A shape.
 * @interface
 */
function Shape() {};
Shape.prototype.draw = function() {};

/**
 * @constructor
 * @implements {Shape}
 */
function Square() {};
Square.prototype.draw = function() {
  ...
};
```

---

### `@interface` `@record`

Marks a function as an interface. An interface specifies the required members of a type. Any class that implements an interface must implement all of the methods and properties defined on the interface's prototype. See `@implements`.

The compiler verifies that interfaces are not instantiated. If the `new` keyword is used with an interface function, the compiler produces a warning.

For the differences between @record and @interface, see [[Structural Interfaces|Structural-Interfaces-in-Closure-Compiler]]

```javascript
/**
 * A shape.
 * @interface
 */
function Shape() {};
Shape.prototype.draw = function() {};

/**
 * A polygon.
 * @interface
 * @extends {Shape}
 */
function Polygon() {};

Polygon.prototype.getSides = function() {};
```

---

### `@lends {objectName}`

Indicates that the keys of an object literal should be treated as properties of some other object. This annotation should only appear on object literals.

Notice that the name in braces is not a type name like in other annotations. It's an object name. It names the object to which the properties are lent. For example, `@type {Foo}` means "an instance of Foo", but `@lends {Foo}` means "the constructor Foo".

The JSDoc Toolkit docs have more information on this annotation.

```javascript
goog.object.extend(
    Button.prototype,
    /** @lends {Button.prototype} */ ({
      isButton: function() { return true; }
    }));
```

---


### `@param {Type} varname` _Description_

Used with method, function and constructor definitions to specify the types of function arguments.

The `@param` tag must be followed by a type expression.

Alternatively, you can annotate the types of the parameters inline (see function foo in the example).

```javascript
/**
 * Queries a Baz for items.
 * @param {number} groupNum Subgroup id to query.
 * @param {string|number|null} term An itemName,
 *     or itemId, or null to search everything.
 */
goog.Baz.prototype.query = function(groupNum, term) {
  ...
};

function foo(/** number */ a, /** number */ b) {
  return a - b + 1;
}
```

---

### `@return {Type}` _Description_

Specifies the return types of method and function definitions. The `@return` tag must be followed by a type expression.

Alternatively, you can annotate the return type inline (see function `foo` in the example).

If a function that is not in externs has no return value, you can omit the `@return` tag, and the compiler will assume that the function returns `undefined`.

```javascript
/**
 * Returns the ID of the last item.
 * @return {string} The hex ID.
 */
goog.Baz.prototype.getLastId = function() {
  ...
  return id;
};

function /** number */ foo(x) { return x - 1; }
```
---

### `@template T`

See [Generic Types](https://github.com/google/closure-compiler/wiki/Generic-Types).

```javascript
/**
 * @param {T} t
 * @constructor
 * @template T
 */
Container = function(t) { ... };
```

---

### `@this {Type}`

Specifies the type of the object to which the keyword this refers within a function. The `@this` tag must be followed by a type expression.

To prevent compiler warnings, you must use a `@this` annotation whenever `this` appears in a function that is neither a prototype method nor a function marked as a `@constructor`.

```javascript
    /**
     * Returns the roster widget element.
     * @this {Widget}
     * @return {Element}
     */
    function() {
      return this.getComponent().getElement();
    };
```

---

### `@type {Type}`

Identifies the type of a variable, property, or expression. The `@type` tag must be followed by a type expression. You can also write the type annotation inline and omit `@type`, as in the second example.

```javascript
/**
 * The message hex ID.
 * @type {string}
 */
var hexId = hexId;
var /** string */ name = 'Jamie';
```

#### Type casting

The Closure Compiler enforces compile-time type casts.

To cast a value to a specific type, use this syntax:

`/** @type {!MyType} */ (valueExpression)`

The parentheses around the expression are required.

Values can only be cast to a subtype or supertype of their actual type. For example, it is forbidden to cast a `string` to `number`.


#### Type expressions

This content is moved to https://github.com/google/closure-compiler/wiki/Types-in-the-Closure-Type-System.

---

### `@typedef {Type}`

Complex types (including unions, and record types) can be aliased for convenience and maintainability using a typedef. These annotations can be long, but can be split over multiple lines for readability.

```js
/** 
 * @typedef {{
 *            foo:string,
 *            bar:number,
 *            foobar: (number|string)
 *          }}
 */
var Mytype;

/** @param {MyType} bar */
function foo(bar) {}
```

---

## Visibility checks

The typechecker auses these tags to help control who can access your code and how they access it.

---

### `@deprecated` _Description_

Warns against using the marked function, method, or property should not be used. Using a deprecated method produces a compiler warning.

```javascript
/**
 * Determines whether a node is a field.
 * @return {boolean} True if the contents of
 *     the element are editable, but the element
 *     itself is not.
 * @deprecated Use isField().
 */
BN_EditUtil.isTopEditableField = function(node) {
  ...
};
```

---

### `@final`

On a class, indicates that this class is not allowed to be extended. On a method, indicates that no subclass is allowed to override that method.

```javascript
/**
 * An old-style class that cannot be extended.
 * @final
 * @constructor
 */
sloth.MyFinalClass = function() { ... }

/**
 * A class that cannot be extended.
 * @final
 */
sloth.MyFinalClass = class { ... }

/**
 * A method that cannot be overridden.
 * @final
 */
sloth.MyFinalClass.prototype.method = function() { ... };
```

---

### `@package`

Marks a member or property as package private. Only code in the same directory can access names marked @package. In particular, code in parent and child directories cannot access names marked `@package`.

Public constructors can have `@package` properties to restrict the methods that callers outside the directory can use. On the other hand, `@package` constructors can have public properties to prevent callers outside the directory from directly instantiating a type.

```javascript
/**
 * Returns the window object the foreign document resides in.
 *
 * @return {Object} The window object of the peer.
 * @package
 */
goog.net.xpc.CrossPageChannel.prototype.getPeerWindowObject = function() {
  // ...
};
```

---

### `@public`

Indicates that a member or property is public. A property marked `@public` is accessible to all code in the any file.  This is the implicit default and rarely used.  This is not used to indicate that name should be preserved in obfuscating builds see `@export`.

```javascript
/**
 * @public
 */
goog.ui.Component.prototype.setElementInternal = function(element) {
  // ...
};
```

---

### `@private`

Marks a member as private. Only code in the same file can access global variables and functions marked `@private`. Constructors marked `@private` can only be instantiated by code in the same file and by their static and instance members.

The public static properties of constructors marked `@private` may also be accessed anywhere.

```javascript
/**
 * Handlers that are listening to this logger.
 * @private {Array<Function>}
 */
this.handlers_ = [];
```

---

### `@protected`

Indicates that a member or property is protected. A property marked `@protected` is accessible to:

1. all code in the same file
1. static methods and instance methods of any subclass of the class on which the property is defined.

```javascript
/**
 * Sets the component's root element to the given element.
 * Considered protected and final.
 * @param {Element} element Root element for the component.
 * @protected
 */
goog.ui.Component.prototype.setElementInternal = function(element) {
  // ...
};
```

--- 

## Other typechecker annotations

---

### `@const`, `@const {type}`

Marks a variable as read-only. The compiler can inline `@const` variables, which optimizes the JavaScript code.

The type declaration is optional.

The compiler produces a warning if a variable marked with `@const` is assigned a value more than once. If the variable is an object, note that the compiler does not prohibit changes to the properties of the object.

```javascript
/** @const */ var MY_BEER = 'stout';  // NOTE: in most cases, you should instead use the const keyword here.

/**
 * My namespace's favorite kind of beer.
 * @const {string}
 */
mynamespace.MY_BEER = 'stout';

class User {
  /** @param {string} beer */
  constructor(beer) {
    /**
     * My user's favorite kind of beer.
     * @const {string}
     */
    this.myBeer = beer;
  }
}
```

The `@const` annotation is also how aliases for interfaces, records, classes and enums are annotated. Without that annotation the alias symbol will not be recognized in a type position.

```javascript
/** @enum {number} */
const E = {...};

/** @const */
const EAlias = E; 
```

Note that repeating the original annotation, e.g. `@enum` at the alias declaration site is not allowed.

```javascript
/** @enum */
const EAlias = E;  // wrong
```

---

### `@dict`

`@dict` is used to create objects with a variable number of properties. When a constructor (Foo in the example) is annotated with `@dict`, you can only use the bracket notation to access the properties of Foo objects. The annotation can also be used directly on object literals.

```javascript
/**
 * @constructor
 * @dict
 */
function Foo() {}
var obj1 = new Foo();
obj1['x'] = 123;
obj1.x = 234;  // warning

var obj2 = /** @dict */ { 'x': 321 };
obj2.x = 123;  // warning
```

or using the class keyword:

```javascript
/** @dict */
class Foo {}
var obj1 = new Foo();
obj1['x'] = 123;
obj1.x = 234;  // warning

var obj2 = /** @dict */ { 'x': 321 };
obj2.x = 123;  // warning
```

---

### `@implicitCast`

This annotation can only appear in externs property declarations. The property has a declared type, but you can assign any type to it without a warning. When accessing the property, you get back a value of the declared type. For example, `element.innerHTML` can be assigned any type, but will always return a string.

```javascript
/**
 * @type {string}
 * @implicitCast
 */
Element.prototype.innerHTML;
```

---

### `@inheritDoc`

Indicates that a method or property of a subclass intentionally hides a method or property of the superclass, and has exactly the same documentation. Note that the `@inheritDoc` tag implies the `@override` tag, `@override` is preferred.

```javascript
/** @inheritDoc */
project.SubClass.prototype.toString = function() {
  ...
};
```

This tag is not required when a class extends a superclass or implements an interface - in those cases the documentation is automatically inherited. This tag is mainly useful to catch typos such as a method named `foobar1` in the superclass and mistakenly named `foobar2` in the subclass.

---

### `@override`

Indicates that a method or property is part of an interface implemented by the class or it intentionally hides a method or property of the superclass. If no other annotations are included, the method or property automatically inherits annotations from its superclass.

```javascript
/**
 * @return {string} Human-readable representation of
 *     project.SubClass.
 * @override
 */
project.SubClass.prototype.toString = function() {
  ...
};
```

Adding or omitting the `@override` tag does not change what type the compiler infers for a method in a class that extends a superclass or implements an interface. The method's documentation is always automatically inherited from the overridden method in those cases. However, the recommended compiler flagset includes warnings for missing and spurious `@override` annotations. These warnings are mainly useful to catch typos, like a method named `foobar1` in the superclass and mistakenly named `foobar2` in the subclass.

---

### `@polymer`

Indicates that a `class` has Polymer semantics. Only affects the compiler when the `--polymer_pass=2` flag is specified. Allows the [Polymer Pass](https://github.com/google/closure-compiler/wiki/Polymer-Pass) to recognize Polymer classes and add appropriate type information.

```javascript
/**
 * A Polymer class
 * @polymer
 */
class MyElement extends Polymer.Element {}
```
---

### `@polymerBehavior`

Indicates that a global object is usable as a Polymer behavior. Allows the [Polymer Pass](https://github.com/google/closure-compiler/wiki/Polymer-Pass) to recognize behaviors and copy type information over to the using Polymer definition. Behavior objects must be global.

```javascript
/** @polymerBehavior */
var FunBehavior = {
  /** @param {string} funAmount */
  doSomethingFun: function(funAmount) { alert('Something fun!'); },
};

var A = Polymer({
  is: 'x-custom',
  behaviors: [ FunBehavior ],
});
```

---

### `@struct`

`@struct` is used to create objects with a fixed number of properties. When a constructor (Foo in the example) is annotated with `@struct`, you can only use the dot notation to access the properties of Foo objects, not the bracket notation. Also, you cannot add a property to a Foo instance after it's constructed. The annotation can also be used directly on object literals.

Classes created with the ES class keyword or with `goog.defineClass` are `@struct` by default, so this annotation is only needed for old-style `@constructor` functions.

```javascript
/**
 * @constructor
 * @struct
 */
function Foo(x) {
  this.x = x;
}

var obj1 = new Foo(123);
var someVar = obj1.x;  // OK
obj1.x = "qwerty";  // OK
obj1['x'] = "asdf";  // warning
obj1.y = 5;  // warning

var obj2 = /** @struct */ { x: 321 };
obj2['x'] = 123;  // warning
```

---

### `@suppress {warningGroup1,warningGroup2}`
Suppresses warnings. Warning categories are separated by | or ,.
For a list of warning names, take a look at [[Warnings|Warnings]].

For example:
```javascript
/**
 * @suppress {deprecated}
 */
function f() {
  deprecatedVersionOfF();
}
```

---

### `@unrestricted`

Indicates that a class is neither a `@struct` type nor a `@dict` type. This is the default for old Closure-style classes created with a `@constructor` function. Both `goog.defineClass` and the ES class keyword produce classes which are `@struct` by default, so will need to be explicitly labeled `@unrestricted`.

```javascript
/**
 * @constructor
 * @unrestricted (This annotation is optional)
 */
function Foo(x) {
  this.x = x;
}

var obj1 = new Foo(123);
var someVar = obj1.x;  // OK
obj1.x = "qwerty";  // OK
obj1['x'] = "asdf";  // OK
obj1.y = 5;  // OK
```

or using the class keyword:

```javascript
/** @unrestricted */
class Foo {
  constructor(x) {
    this.x = x;
  }
}

var obj1 = new Foo(123);
var someVar = obj1.x;  // OK
obj1.x = "qwerty";  // OK
obj1['x'] = "asdf";  // OK
obj1.y = 5;  // OK
```

---

## Other annotations

These annotations are not used by the typechecker. They may either affect the output of the compiler optimizations or provide documentation.

### `@define {Type}` _description_

Indicates a constant that can be overridden by the compiler at compile-time. With the example on the left, you can pass the flag `--define='ENABLE_DEBUG=false'` to the compiler to change the value of `ENABLE_DEBUG` to false. The type of a defined constant can be `number`, `string` or `boolean`. Defines are only allowed in the global scope.

```javascript
/** @define {boolean} */
var ENABLE_DEBUG = true;

/** @define {boolean} */
goog.userAgent.ASSUME_IE = false;
```

---

### `@export`, `@export {SomeType}`

When properties are marked with `@export` and the compiler is run with the `--generate_exports` flag, a corresponding `goog.exportSymbol` statement will be generated:

```javascript
/** @export */
foo.MyPublicClass.prototype.myPublicMethod = function() {
  // ...
};
```

```javascript
goog.exportSymbol('foo.MyPublicClass.prototype.myPublicMethod',
foo.MyPublicClass.prototype.myPublicMethod);
```

You can write `/** @export {SomeType} */` as a shorthand for `/** @export @type {SomeType} */`.

Code that uses the @export annotation must either:

1. include `closure/base.js`, or
1. define both `goog.exportSymbol` and `goog.exportProperty` with the same method signature in their own codebase.

---

### `@externs`

Declares an externs file.

Files annotated with @externs in the @fileoverview block are treated as externs files even if they appear compilation units regular srcs.

For example:
```javascript
/**
 * @fileoverview This is an externs file.
 * @externs
 */

var document;
```

---

### `@fileoverview Description`

Makes the comment block provide file level information including suppressions.

For example:
```javascript
/**
 * @fileoverview Utilities for doing things 
 */
```

---

### `@license`, `@preserve` _Description_

Tells the compiler to insert the associated comment before the compiled code for the marked file. This annotation allows important notices (such as legal licenses or copyright text) to survive compilation unchanged. Line breaks are preserved.

```javascript
/**
 * @preserve Copyright 2009 SomeThirdParty.
 * Here is the full license text and copyright
 * notice for this file. Note that the notice can span several
 * lines and is only terminated by the closing star and slash:
 */
```

---

### `@noalias`

Used in an externs file to indicate to the compiler that the variable or function should not be aliased as part of the "alias externals" pass of the compiler, is not enabled by default and only available through the Java API.

For example:
```javascript
/** @noalias */
function Range() {}
```

---

### `@nocompile`

For example:
```javascript
/** @nocompile */
```

Used at the top of a file to tell the compiler to parse this file but not compile it. Code that is not meant for compilation and should be omitted from compilation tests (such as bootstrap code) uses this annotation.  Most code should use other means such as a `@define` to change behavior.

---

### `@nocollapse`

Denotes a property that should not be collapsed by the compiler into a variable. The primary use for `@nocollapse` is to allow exporting of mutable properties. Note that non-collapsed properties can still be renamed by the compiler. If you annotate a property that is an object with `@nocollapse`, all its properties will also remain uncollapsed.

```javascript
/**
 * A namespace.
 * @const
 */
var foo = {};

/**
 * @nocollapse
 */
foo.bar = 42;

window['foobar'] = foo.bar;
```

---

### `@noinline`

(EXPERIMENTAL) Denotes a function or variable that should not be inlined by the optimizations. 

Currently, this annotation works in the common case, but there may be other code paths in the compiler that still cause inlining. See https://github.com/google/closure-compiler/issues/2751#issuecomment-353484914 for more background and file a feature request if you have another use case for this annotation.

``` js
/** @noinline */
function dontInlineMe() {
  return 0;
}
/** @noinline */
const orMe = 1;
console.log(dontInlineMe() + orMe);
```

---

### `@nosideeffects` `@modifies {this|arguments}`

`@nosideeffects` indicates that a call to the declared function has no side effects. This annotation allows the compiler to remove calls to the function if the return value is not used.  This is not a signal that the function is "pure": it may still read mutable global state.  

`@modifies {this}` signals that only direct properties of the provided `this` are modified.  Calls to these functions may be removed if the "this" object is known to be otherwise unused.

`@modifies {arguments}` signals that only direct properties of the provided arguments are modified.  Calls to these functions may be removed if the parameters provided to the function are otherwise known to be unused.

These annotations are only allowed in extern files.

```javascript
/** @nosideeffects */
function noSideEffectsFn1() {
  ...
};

/** @nosideeffects */
var noSideEffectsFn2 = function() {
  ...
};

/** @nosideeffects */
a.prototype.noSideEffectsFn3 = function() {
  ...
};
```

---

### `@throws {Type}`

Used to document the exceptions thrown by a function. The type checker does not currently use this information. It is only used to figure out if a function declared in an externs file has side effects.

```javascript
/**
 * @throws {DOMException}
 */
DOMApplicationCache.prototype.swapCache = function() { ... };
