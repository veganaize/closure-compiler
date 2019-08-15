# JavaScript Types

The Closure type system was originally based on the
[EcmaScript 4 spec](http://www.google.com/url?sa=D&q=http://wiki.ecmascript.org/doku.php?id=spec:spec).

Closure types always appear in comments, never in the syntax of JavaScript
itself. 

You can specify the data type of any variable, property, expression or function parameter with a type expression. Use a type expression with the `@param` tag to declare the type of a function parameter.
Use a type expression with the `@type` tag to declare the type of a variable, property, or expression. See
[Annotating JavaScript for the Closure Compiler](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler)
for information on what JSDoc tags you can use to annotate your code.

Here's a simple example of using the Closure type system:

```js
/**
 * @param {number} m
 * @param {number} n
 * @return {number}
 */
function add(m, n) {
  return m + n;
}
```

The more types you specify in your code, the more optimizations the compiler can make and the more mistakes it can catch.

The compiler uses these annotations to type-check your program.

Note that the Closure Compiler does not make any promises that it will be able to figure out the type of every expression in your program.
It makes a best effort by looking at how variables are used, and at the type annotations attached to their declarations. Then, it uses a number of heuristics to figure out the type of as many expressions as possible.

Some of these heuristics are straightforward ("if `x` is a number, and we see `y = x;`, then `y` is a number").
Some are more indirect ("if `f`'s first parameter is documented as a callback that must take a number, and we see `f(function(x) { /** ... */ });`, then `x` must be a number").

## The JavaScript Type Language

The ES4 proposal contained a language for specifying JavaScript types. We use
this language in JsDoc to express the types.

Syntax Name                                                       | Syntax                                                                                                                                                                                                                                                                                                                                                          | Description
----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -----------
Primitive Type                                                    | There are 6 primitive types in JavaScript: `null`, `undefined`, `boolean`, `number`, `string` and `symbol` .                                                                                                                                                                                                                                                    | Simply the name of a type.<br />The primitive types (other than `null`) are not nullable.
Instance Type                                                     | `Object` : An instance of Object, or null. <br/>`Function` : An instance of Function, or null.<br/> `EventTarget` : An instance of a constructor that implements the EventTarget interface, or null.                                                                                                                                                            | An instance of a constructor or interface function.<br/>Constructor functions are functions defined with the `@constructor` JSDoc tag or classes. Interface functions are functions or classes annotated with [`@interface` or `@record`](https://github.com/google/closure-compiler/wiki/Structural-Interfaces-in-Closure-Compiler).<br/> By default, instance types will accept null. Whenever possible, [avoid using`Object` in favor of a more specific existing type](https://github.com/google/closure-compiler/wiki/A-word-about-the-type-Object).
Enum Type                                                         | `goog.events.EventType` <br/>One of the properties of the object literal initializer of`goog.events.EventType`.                                                                                                                                                                                                                                                 | An enum must be initialized as an object literal, or as an alias of another enum, annotated with the`@enum` JSDoc tag. The properties of this literal are the instances of the enum. The syntax of the enum is defined [here](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler#enum-type).<br/><br />Nullablity of the enum value depends on the referenced type.`@enum {string}` or`@enum {number}` is not nullable by default, while`@enum {Object}` is.
Type Application                                                  | `?Array<string>`: A nullable array of strings.<br/>`!Object<string, number>`: A non-null object in which the keys are strings and the values are numbers.<br/>`!Set<!YourType>`: A non-null Set of non-null instances of YourType.                                                                                                                              | Parameterizes a type, by applying a set of type arguments to that type. The idea is analogous to generics in Java.<br />Deprecated syntax: adding a dot before the`<` (e.g. `!Array.<string>`) is also accepted.
Type Union                                                        | `(number\|boolean)`<br/>A number or a boolean.<br /><br/>Deprecated syntax: <br/>`(number,boolean)`, <br/>`(number\|\|boolean)`                                                                                                                                                                                                                                 | Indicates that a value might have type A OR type B.The parentheses may be omitted at the top-level expression, but the parentheses should be included in sub-expressions to avoid ambiguity: `function(): (number\|boolean)` vs `function(): number\|boolean`
Nullable type                                                     | `?number` A number or null. <br/>Deprecated syntax: <br/>`number?`                                                                                                                                                                                                                                                                                              | Shorthand for the union of the null type with any other type. This is just syntactic sugar. <br/>Note that the following are already nullable, and thus prepending`?` is redundant, but is recommended so that the intent is clear and explicit: <br/>`?Object`, `?Array`, `?Function`
Non-nullable type                                                 | `!Object` <br/>An Object, but never the `null` value. <br/>Deprecated syntax: <br/>`Object!`                                                                                                                                                                                                                                                                    | Filters null out of nullable types. Most often used with instance types, which are nullable by default. <br/>Note that the following are already non-nullable, and thus prepending`!` is redundant: <br/>`!number`, `!string,` `!boolean`, `!{foo: string}`, `!function()`
Record Type                                                       | `{myNum: number, myObject}` <br/>An anonymous type with the given type members.                                                                                                                                                                                                                                                                                 | Indicates that the value has the specified members with the specified types. In this case, `myNum` with a type `number` and `myObject` with any type. <br/> Notice that the braces are part of the type syntax. For example, to denote an `Array` of objects that have a`length` property, you might write `Array<{length}>`. Record types are not nullable.
Function Type                                                     | `function(string, boolean)` <br/> A function that takes two arguments (a `string` and a `boolean`), and has an unknown return value.                                                                                                                                                                                                                            | Specifies a function.<br/>Also note the difference between `function()` and`Function`. The latter is an instance type and is nullable by default.`function(...)` should be used instead of`Function` whenever possible because it provides more type information about its parameters and return value.
Function Return Type                                              | `function(): number` <br/>A function that takes no arguments and returns a number.                                                                                                                                                                                                                                                                              | Specifies a function return type.
Function `this` Type                                              | `function(this:goog.ui.Menu, string)` <br/>A function that takes one argument (a string), and executes in the context of a goog.ui.Menu.                                                                                                                                                                                                                        | Specifies the context type of a function type.
Function `new` Type                                               | `function(new:goog.ui.Menu, string)` <br/>A constructor that takes one argument (a string), and creates a new instance of `goog.ui.Menu` when called with the `new` keyword.                                                                                                                                                                                    | Specifies the constructed type of a constructor.
Variable arguments                                                | `function(string, ...number): number` <br/>A function that takes one argument (a string), and then a variable number of arguments that must be numbers.                                                                                                                                                                                                         | Specifies variable arguments to a function. <br/> Nullability of the arguments is determined by the type annotation after the `...`
Variable arguments (in`@param` annotations)                       | `@param {...number} var_args` <br/>A variable number of arguments to an annotated function. Rest parameters (`function f(...args) {}`) are implicitly considered to be variable, even if not annotated with the `...`.                                                                                                                                                                                                                                                                    | Specifies that the annotated function accepts a variable number of arguments. Nullability of the arguments is determined by the type annotation after the `...`
Function [optional arguments](#optional)                          | `function(?string=, number=)` <br/>A function that takes one optional, nullable string and one optional number as arguments. The`=` syntax is only for`function` type declarations. <br/>`function(...)` is not nullable. Nullability of arguments is determined by the unadorned type annotation. See [nullable vs. optional](#optional) for more information. | Specifies optional arguments to a function.
Function [optional arguments](#optional) (in`@param` annotations) | `@param {number=} size` <br/>An optional parameter of type `number`.                                                                                                                                                                                                                                                                                    | Specifies that the annotated function accepts an optional argument.
Typeof operator                                                   | `typeof ns` <br/>The type of the namespace `ns`.                                                                                                                                                                                                                                                                                                                | Evaluates to the type of a given value, which must be constant and declared (rather than inferred). Allows expressing the type of namespaces, constructors, and enum namespaces.
The ALL type                                                      | `*`                                                                                                                                                                                                                                                                                                                                                             | Indicates that the variable can take on any type.
The ANY type                                                      | `?`                                                                                                                                                                                                                                                                                                                                                             | Indicates that the variable can take on any type, and the compiler should not type-check any uses of it.

## Types in JavaScript

Type Example                                          | Value Examples                                                                                                                                                              | Description
----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -----------
number                                                | `1, 1.0, -5, 1e5, Math.PI`                                                                                                                                                  |
Number                                                | `new Number(true)`                                                                                                                                                          | [Number object](#Wrapper_objects_for_primitive_types)
string                                                | `'Hello', "World", String(42)`                                                                                                                                              | String value
String                                                | `new String('Hello'),new String(42)`                                                                                                                                        | [String object](#Wrapper_objects_for_primitive_types)
boolean                                               | `true,false,Boolean(0)`                                                                                                                                                     | Boolean value
Boolean                                               | `new Boolean(true)`                                                                                                                                                         | [Boolean object](#Wrapper_objects_for_primitive_types)
RegExp                                                | `new RegExp('hello'), /world/g`                                                                                                                                             |
Date                                                  | `new Date, new Date()`                                                                                                                                                      |
_preferred:_ null <br />_deprecated:_ Null            | `null`                                                                                                                                                                      |
_preferred:_ undefined <br /> _deprecated:_ Undefined | `undefined`                                                                                                                                                                 |
void                                                  | `function f() { return; }`                                                                                                                                                  | No return value
Array                                                 | `['foo', 0.3, null]`,`[]`                                                                                                                                                   | Untyped Array
Array<number>                                         | `[11, 22, 33]`                                                                                                                                                              | An Array of numbers
Array<Array<string>>                                  | `[ ['one', 'two', 'three'], ['foo', 'bar']]`                                                                                                                                | Array of Arrays of strings
Object                                                | `{}`,<br />`{`<br />`foo: 'abc',`<br />`bar: 123,`<br />`baz: null`<br />`}`                                                                                                |
Object<string>                                        | `{'foo': 'bar'}`                                                                                                                                                            | An Object in which the values are strings.
Object<number, string>                                | `var obj = {};`<br />`obj[1] = 'bar';`                                                                                                                                      | An Object in which the keys are numbers and the values are strings.Note that in JavaScript, the keys are always implicitly converted to strings, so `obj['1'] == obj[1]`. So the key will always be a string in for...in loops. But the compiler will verify the type of the key when indexing into the object.
Function                                              | `function(x, y) { return x * y; }`                                                                                                                                          | [Function object](#Wrapper_objects_for_primitive_types)
function(number, number): number                      | `function(x, y) { return x * y; }`                                                                                                                                          | function value
constructor                                           | `class C {} new C()` <br /> `/** @constructor */` function D() {}                                                                                                               | constructors are created when you declare a function `@constructor`, or when you create an ES6 class. Only constructors can be new'd.
interface                                             | `/** @interface */`<br />`class I {`<br />`draw() {}`<br />`}`                                                                                                              | See [Nominal Interfaces](https://github.com/google/closure-compiler/wiki/Structural-Interfaces-in-Closure-Compiler#nominal-interfaces)
structural interface                                                | `/** @record */`<br />`class R {}`                            | Like an interface, but is checked using structural equality only. Any value with matching properties is convertible to the record type. See [Structural Interfaces in Closure Compiler](https://github.com/google/closure-compiler/wiki/Structural-Interfaces-in-Closure-Compiler#structural-interfaces)
enum                                                  | `/** @enum {string} */`<br />`project.MyEnum = {`<br /> `/** The color blue. */`<br />`BLUE:<br />'#0000dd',`<br /> `/** The color red. */`<br />`RED: '#dd0000'`<br />`};` | JSDoc comments on the enum values are optional.
Element                                               | document.createElement('div')                                                                                                                                               | Elements in the DOM.
Node                                                  | document.body.firstChild                                                                                                                                                    | Nodes in the DOM.

## Type Casts

In cases where type-checking doesn't accurately infer the type of an expression,
it is possible to add a type cast comment by adding a type annotation comment
and enclosing the expression in parentheses. The parentheses are required.

```js
/** @type {number} */ (x)
```

## Optional

Function parameters may be marked optional with the use of `=`:

``` js
/** 
 * @param {number} height
 * @param {number=} width
 */
function f(height, width) {}
f(10, 10);  // Ok
f(10);      // Ok, width is optional
f();        // Not ok, height is required.
```

Note that any parameter with a default value is considered optional, even without explicit JSDoc.

``` js
function f(height, width = 0) {}
f(10, 10);  // Ok
f(10);      // Ok, width is optional
f();        // Not ok, height is required.
```

## Nullable vs. Optional Parameters and Properties

Because JavaScript is a loosely-typed language, it is very important to
understand the subtle differences between optional, nullable, and undefined
function parameters and class properties.

Instances of classes and interfaces are nullable by default. For example, the
following declaration

```js
/**
 * Some class, initialized with a value.
 * @param {Object} value Some value.
 * @constructor
 */
function MyClass(value) {
  /**
   * Some value.
   * @private {Object}
   */
  this.myValue_ = value;
}
```

tells the compiler that the `myValue_` property holds either an Object or null.
If `myValue_` must never be null, it should be declared like this:

```js
/**
 * Some class, initialized with a non-null value.
 * @param {!Object} value Some value.
 * @constructor
 */
function MyClass(value) {
  /**
   * Some value.
   * @private {!Object}
   */
  this.myValue_ = value;
}
```

This way, if the compiler can determine that somewhere in the code `MyClass` is
initialized with a null value, it will issue a warning.

You may see type declarations like these in older code:

```js
@type {Object?}
@type {Object|null}
```

Optional parameters to functions may be undefined at runtime, so if they are
assigned to class properties, those properties must be declared accordingly:

```js
/**
 * Some class, initialized with an optional value.
 * @param {!Object=} opt_value Some value (optional).
 * @constructor
 */
function MyClass(opt_value) {
  /**
   * Some value.
   * @private {!Object|undefined}
   */
  this.myValue_ = opt_value;
}
```

This tells the compiler that `myValue_` may hold an Object, or remain undefined.

Note that the optional parameter `opt_value` is declared to be of
type`{!Object=}`, not `{!Object|undefined}`. This is because optional parameters
may, by definition, be undefined. While there is no harm in explicitly declaring
an optional parameter as possibly undefined, it is both unnecessary and makes
the code harder to read.

Finally, note that being nullable and being optional are orthogonal properties.
The following four declarations are all different:

```js
/**
 * Takes four arguments, two of which are nullable, and two of which are
 * optional.
 * @param {!Object} nonNull Mandatory (must not be undefined), must not be null.
 * @param {?Object} mayBeNull Mandatory (must not be undefined), may be null.
 *     ({Object} would mean the same thing, but is not as explicit.)
 * @param {!Object=} opt_nonNull Optional (may be undefined), but if present,
 *     must not be null!
 * @param {?Object=} opt_mayBeNull Optional (may be undefined), may be null.
 *     ({Object=} would mean the same thing, but is not as explicit.)
 */
function strangeButTrue(nonNull, mayBeNull, opt_nonNull, opt_mayBeNull) {
  // ...
};
```

## Typedefs

Sometimes types can get complicated. A function that accepts content for an
Element might look like:

```js
/**
 * @param {string} tagName
 * @param {(string|Element|Text|Array<Element>|Array<Text>)} contents
 * @return {!Element}
 */
goog.createElement = function(tagName, contents) {
  ...
};
```

You can define commonly used type expressions with a `@typedef` tag. For
example,

```js
/** @typedef {(string|Element|Text|Array<Element>|Array<Text>)} */
goog.ElementContent;

/**
 * @param {string} tagName
 * @param {goog.ElementContent} contents
 * @return {!Element}
 */
goog.createElement = function(tagName, contents) {
...
};
```

## Template types

Functions and classes can be annotated with template types using `@template T, U`. 
These type work similarly to Java's generic types. For example:

``` js
/** @template T */
class Wrapper {
  /** @param {T} item */
  constructor(item) {
    /** @const */
    this.item = item;
  }
}
/** @param {!Wrapper<!Array<string>>} wrappedArray */
function f(wrappedArray) {
  console.log(wrappedArray.item.length);
}
f(new Wrapper(['foo', 'bar'])); // Ok
f(new Wrapper(0));  // Bad
f([]);  // Bad
```

The compiler has limited support for inferring template types. It can only infer the type
of`this` inside an anonymous function literal from the type of the`this`
argument and whether the `this` argument is missing.

``` js
/**
 * @param {function(this:T, ...)} fn
 * @param {T} thisObj
 * @param {...*} var_args
 * @template T
 */
goog.bind = function(fn, thisObj, var_args) {
...
};
// Possibly generates a missing property warning.
goog.bind(function() { this.someProperty; }, new SomeClass());
// Generates an undefined this warning.
goog.bind(function() { this.someProperty; });
```

Note: the compiler cannot infer the type of array literals! This means that, for example, this code is allowed:
``` js
const arrNum = [0, 1, 2];
/** @type {!Array<string>} */
const arrString = arrNum;
```

### Wrapper objects for primitive types

The JavaScript language has wrappers for primitive types. For example, `new Number(0)` vs `0`. The Closure Compiler also supports both wrapper object types and primitives:
``` js
/** @type {!Number} */
let n = 0;  // Ok
let n = new Number(1);  // Ok
```

The Closure Compiler also includes wrapper types for `Object` and `Function`. The `Object` type matches all types except for primitives:
``` js
/** @type {!Object} */
let o = {};  // Ok
o = [];  // Ok, arrays are objects.
o = function() {};  // Ok, functions are objects.
o = 0;  // Not ok
o = null; // Not ok
```

And the `Function` type matches all functions.

In general, prefer using a more specific type than `Object` or `Function` when possible. See also https://github.com/google/closure-compiler/wiki/A-word-about-the-type-Object.