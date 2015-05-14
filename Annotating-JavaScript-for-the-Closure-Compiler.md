# Overview

> (copied from https://developers.google.com/closure/compiler/docs/js-for-compiler, included in the GitHub wiki to permit community editing)

The Closure Compiler can use data type information about JavaScript variables to provide enhanced optimization and warnings. JavaScript, however, has no way to declare types.

Because JavaScript has no syntax for declaring the type of a variable, you must use comments in the code to specify the data type.

The Closure Compiler's type language derives from the annotations used by the JSDoc document-generation tool. This document describes the set of annotations and type expressions that the Closure Compiler understands.

# JSDoc Tags

The Closure Compiler looks for type information in JSDoc tags. Use the JSDoc tags described in the reference table below to help the compiler optimize your code and check it for possible type errors and other mistakes.

This table includes only tags that affect on the behavior of the Closure Compiler. For information about other JSDoc tags see the JSDoc Toolkit documentation.

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

### `@constructor`

Marks a function as a constructor.  The compiler requires a `@constructor` annotation for any function that is used with the `new` keyword

```javascript
/**
 * A rectangle.
 * @constructor
 */
function GM_Rect() {
  ...
}
```

### `@define {Type}` _description_

Indicates a constant that can be overridden by the compiler at compile-time. With the example on the left, you can pass the flag `--define='ENABLE_DEBUG=false'` to the compiler to change the value of `ENABLE_DEBUG` to false. The type of a defined constant can be `number`, `string` or `boolean`. Defines are only allowed in the global scope.

```javascript
/** @define {boolean} */
var ENABLE_DEBUG = true;

/** @define {boolean} */
goog.userAgent.ASSUME_IE = false;
```

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

### `@enum {Type}`

Specifies the type of an enum. An enum is an object whose properties constitute a set of related constants. The `@enum` tag must be followed by a type expression.  If the type of an enum is omitted, number is assumed.

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


### `@extends {Type}`

Marks a class or interface as inheriting from another class. A class marked with `@extends` must also be marked with either `@constructor` or `@interface`.

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

### `@inheritDoc`

Indicates that a method or property of a subclass intentionally hides a method or property of the superclass, and has exactly the same documentation. Note that the `@inheritDoc` tag implies the `@override` tag.

```javascript
/** @inheritDoc */
project.SubClass.prototype.toString = function() {
  ...
};
```

### `@interface`

Marks a function as an interface. An interface specifies the required members of a type. Any class that implements an interface must implement all of the methods and properties defined on the interface's prototype. See `@implements`.

The compiler verifies that interfaces are not instantiated. If the `new` keyword is used with an interface function, the compiler produces a warning.

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

### `lends {objectName}`

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

### `@nosideeffects`

Indicates that a call to the declared function has no side effects. This annotation allows the compiler to remove calls to the function if the return value is not used. The annotation is only allowed in extern files.

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

### `@override`

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

Indicates that a method or property of a subclass intentionally hides a method or property of the superclass. If no other annotations are included, the method or property automatically inherits annotations from its superclass.

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

### `@private`

Marks a member as private. Only code in the same file can access global variables and functions marked `@private`. Constructors marked `@private` can only be instantiated by code in the same file and by their static and instance members.

The public static properties of constructors marked `@private` may also be accessed anywhere, and the `instanceof` operator can always access `@private` members.

```javascript
/**
 * Handlers that are listening to this logger.
 * @private {Array<Function>}
 */
this.handlers_ = [];
```


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

### `@return {Type}` _Description_

Specifies the return types of method and function definitions. The `@return` tag must be followed by a type expression.

If there is no return value, do not use a `@return` tag.

Alternatively, you can annotate the return type inline (see function `foo` in the example).

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

### `@template T`

See Generic Types.

```javascript
/**
 * @param {T} t
 * @constructor
 * @template T
 */
Container = function(t) { ... };
```

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

### `@throws {Type}`

Used to document the exceptions thrown by a function. The type checker does not currently use this information. It is only used to figure out if a function declared in an externs file has side effects.

```javascript
/**
 * @throws {DOMException}
 */
DOMApplicationCache.prototype.swapCache = function() { ... };
```

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

### `@typedef {Type}`

Declares an alias for a more complex type.

```javascript
/** @typedef {(string|number)} */
goog.NumberLike;

/** @param {goog.NumberLike} x A number or a string. */
goog.readNumber = function(x) {
  ...
}
```

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