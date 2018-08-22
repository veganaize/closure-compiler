# Overview

> (copied from https://developers.google.com/closure/compiler/docs/js-for-compiler, included in the GitHub wiki to permit community editing)

The Closure Compiler can use data type information about JavaScript variables to provide enhanced optimization and warnings. JavaScript, however, has no way to declare types.

Because JavaScript has no syntax for declaring the type of a variable, you must use comments in the code to specify the data type.

The Closure Compiler's type language derives from the annotations used by the JSDoc document-generation tool. This document describes the set of annotations and type expressions that the Closure Compiler understands.

# JSDoc Tags

The Closure Compiler looks for type information in JSDoc tags. Use the JSDoc tags described in the reference table below to help the compiler optimize your code and check it for possible type errors and other mistakes.

This table includes only tags that affect on the behavior of the Closure Compiler. For information about other JSDoc tags see the JSDoc Toolkit documentation.

---

### `@const`, `@const {type}`

Marks a variable as read-only. The compiler can inline `@const` variables, which optimizes the JavaScript code.

The type declaration is optional.

The compiler produces a warning if a variable marked with `@const` is assigned a value more than once. If the variable is an object, note that the compiler does not prohibit changes to the properties of the object.

```javascript
/** @const */ var MY_BEER = 'stout';

/**
 * My namespace's favorite kind of beer.
 * @const {string}
 */
mynamespace.MY_BEER = 'stout';
```

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

### `@define {Type}` _description_

Indicates a constant that can be overridden by the compiler at compile-time. With the example on the left, you can pass the flag `--define='ENABLE_DEBUG=false'` to the compiler to change the value of `ENABLE_DEBUG` to false. The type of a defined constant can be `number`, `string` or `boolean`. Defines are only allowed in the global scope.

```javascript
/** @define {boolean} */
var ENABLE_DEBUG = true;

/** @define {boolean} */
goog.userAgent.ASSUME_IE = false;
```

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

### `@final`

Indicates that this class is not allowed to be extended. For methods, indicates that no subclass is allowed to override that method.

```javascript
/**
 * A class that cannot be extended.
 * @final
 * @constructor
 */
sloth.MyFinalClass = function() { ... }

/**
 * A method that cannot be overridden.
 * @final
 */
sloth.MyFinalClass.prototype.method = function() { ... };
```

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

### `@nosideeffects` `@modifies {this|arguments}`

`@nosideeffects` indicates that a call to the declared function has no side effects. This annotation allows the compiler to remove calls to the function if the return value is not used.  This is not a signal that the function is "pure", it may still read mutable global state.  

`@modifies {this}` signals that only direct properties of the provided `this` are modified.  Calls to these functions may be removed if the "this" object is known to be otherwise unused.

`@modifies {arguments}` signals that only direct properties of the provided arguments are modified.  Calls to these functions may be removed if the parameters provided to the function are otherwise known to be otherwise unused.

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

### `@override`

Indicates that a method or property is part of an implemented by the class or itintentionally hides a method or property of the superclass. If no other annotations are included, the method or property automatically inherits annotations from its superclass.

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

This tag is not required when a class extends a superclass or implements an interface - in those cases the documentation is automatically inherited. This tag is mainly useful to catch typos such as a method named `foobar1` in the superclass and mistakenly named `foobar2` in the subclass.

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

### `@struct`

`@struct` is used to create objects with a fixed number of properties. When a constructor (Foo in the example) is annotated with `@struct`, you can only use the dot notation to access the properties of Foo objects, not the bracket notation. Also, you cannot add a property to a Foo instance after it's constructed. The annotation can also be used directly on object literals.

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

To prevent compiler warnings, you must use a `@this` annotation whenever this appears in a function that is neither a prototype method nor a function marked as a `@constructor`.

```javascript
    /**
     * Returns the roster widget element.
     * @this {Widget}
     * @return {Element}
     */
    function() {
      return this.getComponent().getElement();
    });
```

---

### `@throws {Type}`

Used to document the exceptions thrown by a function. The type checker does not currently use this information. It is only used to figure out if a function declared in an externs file has side effects.

```javascript
/**
 * @throws {DOMException}
 */
DOMApplicationCache.prototype.swapCache = function() { ... };
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

---

### `@typedef {Type}`

Declares an alias for a more complex type.
Currently, typedefs can only be defined at the top level, not inside functions.
We have fixed this limitation in the [new type inference](https://github.com/google/closure-compiler/wiki/Using-NTI-(new-type-inference)).

```javascript
/** @typedef {(string|number)} */
goog.NumberLike;

/** @param {goog.NumberLike} x A number or a string. */
goog.readNumber = function(x) {
  ...
}
```

---

### `@unrestricted`

Indicates that a class is neither a `@struct` type, nor a `@dict` type. This is the default so it is generally not necessary to write it explicitly, unless you are using `goog.defineClass`, or the class keyword, which both produce classes which are `@structs` by default.

```javascript
/**
 * @constructor
 * @unrestricted
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

## Special purpose annotations

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

# Type Expressions

You can specify the data type of any variable, property, expression or function parameter with a type expression. Use a type expression with the `@param` tag to declare the type of a function parameter.
Use a type expression with the `@type` tag to declare the type of a variable, property, or expression.

The more types you specify in your code, the more optimizations the compiler can make and the more mistakes it can catch.

The compiler uses these annotations to type-check your program.
Note that the Closure Compiler does not make any promises that it will be able to figure out the type of every expression in your program.
It makes a best effort by looking at how variables are used, and at the type annotations attached to their declarations.
Then, it uses a number of heuristics to figure out the type of as many expressions as possible.
Some of these heuristics are straightforward ("if `x` is a number, and we see `y = x;`, then `y` is a number").
Some are more indirect ("if `f`'s first parameter is documented as a callback that must take a number, and we see `f(function(x) { /** ... */ });`, then `x` must be a number").

The possible type expressions are listed below. For more information about specific types that can be used in @type annotations see: [Types in the Closure Type System](https://github.com/google/closure-compiler/wiki/Types-in-the-Closure-Type-System).

---

### Type Name

Examples: `boolean`, `Window`, `goog.ui.Menu`

Specifies the name of a type.

---

### Type Application

Examples: `Array<string>` (an array of strings), `Object<string, number>` (an object whose keys are strings and the values are numbers)

Parameterizes a type with a set of type arguments. Similar to Java generics.

---

### Type Union

Example: `(number|boolean)`  
A number or a boolean.
Note the parentheses, which are required.

Indicates that a value might have type `A` OR type `B`.

---

### Record Type

Example: `{myNum: number, myObject}`  
An anonymous type with both a property named `myNum` that has a value of type `number` and a property named `myObject` that has a value of any type.

Indicates that the value has the specified members with values of the specified types.

Braces are part of the record type's syntax.
For example, to denote an Array of objects that have a length property, you might write: `Array<{length}>`.

---

### Nullable type

Example: `?number`  
A number or null.
	
Indicates that a value is type `A` or `null`.

All object types are nullable by default whether or not they are declared with the Nullable operator.
An object type is defined as anything except a function, string, number, or boolean.
To make an object type non-nullable, use the Non-nullable operator.

---

### Non-nullable type

Example: `!Object`  
An `Object`, but never the `null` value.

Indicates that a value is type `A` and not `null`.

Functions and all scalar types (boolean, number, and string) are non-nullable by default whether or not they are declared with the Non-nullable operator.
To make a scalar or function type nullable, use the Nullable operator.

---

### Function Type

Example: `function(string, boolean)`  
A function that takes two parameters (a string and a boolean), and has an unknown return value.

Specifies a function and the types of the function's parameters.

---

### Function Return Type

Example: `function(): number`  
A function that takes no parameters and returns a number.

Specifies the type of a function's return value.

---

### Function `this` Type

Example: `function(this:goog.ui.Menu, string)`  
A function that takes one parameter (a string), and executes in the context of a `goog.ui.Menu`.

Specifies the type of the value of `this` within the function.

---

### Function `new` Type

Example: `function(new:goog.ui.Menu, string)`  
A function that takes one parameter (a string), and creates a new instance of `goog.ui.Menu` when called with the 'new' keyword.

Specifies the constructed type of a constructor.

---

### Variable parameters

Example: `function(string, ...number): number`  
A function that takes one parameter (a string), and then a variable number of parameters that must be numbers.

Indicates that a function type takes a variable number of parameters, and specifies a type for the variable parameters.

---

### Variable parameters (in `@param` annotations)

Example: `@param {...number} var_args`  
A variable number of parameters to an annotated function.

Indicates that the annotated function accepts a variable number of parameters, and specifies a type for the variable parameters.

---

### Optional parameter in a `@param` annotation

Example: `@param {number=} opt_argument`  
An optional parameter of type number.

Indicates that the argument described by a `@param` annotation is optional.
A function call can omit an optional argument.
An optional parameter cannot precede a non-optional parameter in the parameter list.

If a method call omits an optional parameter, that argument will have a value of undefined.
Therefore if the method stores the parameter's value in a class property, the type declaration of that property must include a possible value of undefined, as in the following example:

```javascript
/**
 * Some class, initialized with an optional value.
 * @param {Object=} opt_value Some value (optional).
 * @constructor
 */
function MyClass(opt_value) {
  /**
   * Some value.
   * @type {Object|undefined}
   */
  this.myValue = opt_value;
}
```

---

### Optional argument in a function type

Example: `function(?string=, number=)`  
A function that takes one optional, nullable string and one optional number as arguments.

Indicates that an argument in a function type is optional.
An optional argument can be omitted from the function call.
An optional argument cannot precede a non-optional argument in the argument list.

---

### The ALL type

Example: `*`

Indicates that the variable can take on any type.
The ALL type is a supertype of every other type.
To use a value of type ALL as a more specific type, you need to downcast it first.

---

### The UNKNOWN type

Example: `?`

Indicates that the variable can take on any type, and the compiler should not type-check any uses of it.

---

# Type Casting

To cast a value to a specific type use this syntax

`/** @type {!MyType} */ (valueExpression)`

The parentheses around the expression are required.