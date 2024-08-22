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
Non-nullable type                                                 | `!Object` <br/>An Object, but never the `null` value. <br/>Deprecated syntax: <br/>`Object!`                                                                                                                                                                                                                                                                    | Filters null out of nullable types. Most often used with instance types, which are nullable by default. <br/>Note that the following are already non-nullable, and thus prepending`!` is redundant: <br/>`!number`, `!string,` `!boolean`, `!{foo: string}`, `!function()`, `!bigint
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
The ALL type                                                      | `*`                                                                                                                                                                                                                                                                                                                                                             | Indicates that the variable can take on any type. However, it is an error to attempt to do operations on a value of this type or access any properties on it. You also cannot assign it to any other type variable without a cast.
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
Array\<number\>                                       | `[11, 22, 33]`                                                                                                                                                              | An Array of numbers
Array\<Array\<string\>\>                              | `[ ['one', 'two', 'three'], ['foo', 'bar']]`                                                                                                                                | Array of Arrays of strings
Object                                                | `{}`,<br />`{`<br />`foo: 'abc',`<br />`bar: 123,`<br />`baz: null`<br />`}`                                                                                                |
Object\<string\>                                      | `{'foo': 'bar'}`                                                                                                                                                            | An Object in which the values are strings.
Object\<number, string\>                              | `var obj = {};`<br />`obj[1] = 'bar';`                                                                                                                                      | An Object in which the keys are numbers and the values are strings.Note that in JavaScript, the keys are always implicitly converted to strings, so `obj['1'] == obj[1]`. So the key will always be a string in for...in loops. But the compiler will verify the type of the key when indexing into the object.
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

The Closure Compiler supports 'template types', which are similar to Java's generic types. See https://github.com/google/closure-compiler/wiki/Generic-Types for more details.

## Wrapper objects for primitive types

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

## Union types

Type unions in Closure are often useful, but have a few potential gotchas.

First, in Closure, a type union is considered to have a property `x` if any of the types in the union have such a property. For example, given a union (`Date|Array<string>`), Closure will treat that union as having the `length` property from `Array<string>`. This is despite the fact that `Date`s do not have a `length`.

Second, Closure is very bad about handling unions of function types. The type system has zero support for function overloads. In the worst case, unions of function types are simplified to the very general `Function` type and lose all typechecking. In order to get the most accurate typechecking, never write unions of function literal types.

Bad type information (but better documentation):
``` js
/** @typedef {function(string): string|function(number): number} */
let MyType;
```

Better type information:
``` js
/** @typedef {function(string|number):(string|number)} */
let MyType;
```

## Declared versus inferred types

The Closure type system is designed to be optional: you don't have to add `@type` annotations everywhere. This leads to a distinction in the type system between variables and properties with **declared** versus **inferred** types.

The basic idea is not too hard to grasp. Declared types make the compiler enforce that a variable is only assigned a particular type. Inferred types do not.

``` js
// Example of inferred type
let x = 0;
x = 'a string';  // x has only an inferred type of number before this assignment, so no type warning.

// Example of declared type
/** @type {number} */
let y = 0;
y = 'a string';  // x has a declared type of number, so this causes a warning.
```

### JSDoc type declarations

Here are some examples of declarations:
``` js
/** @type {number} */
let x = 0;  // Note that @export or @const also work
/** @param {string} */
let fn = createStringFn();
/** @enum {string} */
const COLORS = {RED: 0, BLUE: 1};
/** @constructor */
function Klazz() {} // The pre-ES6 style of declaring classes
/** @constructor */
let MyCtor = mixin(BaseCtor); // @constructor works even if not assigning a function literal
```

Not all JSDoc makes a variable declared. JSDoc with a non-type-related annotation (like `@nocollapse`) or with no type annotation (`/** some descriptive comment */`) does not affect whether the variable it is attached to is inferred or declared.

### Non-JSDoc declarations

There are a few cases where a variable declaration or property write without type-related JSDoc is still treated as having a declared type. These include:
 - [functions](https://closure-compiler-debugger.appspot.com/#input0%3Dlet%2520x%2520%253D%2520function(x%252C%2520y)%2520%257B%257D%253B%250A%250Ax%2520%253D%25200%253B%2520%252F%252F%2520type%2520mismatch!%26input1%26conformanceConfig%26externs%26refasterjs-template%26CHECK_TYPES%3Dtrue%26CLOSURE_PASS%3Dtrue%26PRESERVE_TYPE_ANNOTATIONS%3Dtrue%26PRETTY_PRINT%3Dtrue) (`let x = function() {};`, `function f() {}`, or `a.b = function() {}`)
 - [classes](https://closure-compiler-debugger.appspot.com/#input0%3Dlet%2520Klazz%2520%253D%2520class%2520%257B%257D%253B%250A%250AKlazz%2520%253D%25200%253B%2520%2520%252F%252F%2520type%2520mismatch!%26input1%26conformanceConfig%26externs%26refasterjs-template%26CHECK_TYPES%3Dtrue%26CLOSURE_PASS%3Dtrue%26PRESERVE_TYPE_ANNOTATIONS%3Dtrue%26PRETTY_PRINT%3Dtrue) (`let Klazz = class {};`, `class Klazz {}`, or `a.Klazz = class {};`)
 - some (but not all) constant declarations: (`const x = 0;`). This is discussed in more detail below.

### Constant declarations

A variable or property that is a) declared constant and b) is assigned an 'easily inferrable value' will have a declared type. Examples:

``` js
const x = 0;  // declared type of number
/** @const */
var y = 0;  // declared type of number
/** @const */
goog.Number = 0;  // declared type of number
```

Note that all `goog.module` exports are implicitly constant:
``` js
goog.module('my.mod');
exports.x = 0;  // declared type of number
```

The definition of an "easily inferrable value" is, unfortunately, defined only by the behavior of the compiler. Currently things that are 'easily inferrable' include:

 - numeric literals (`const x = 0;`)
 - string or template literals (`const x = '0';`)
 - object literals (`/** @const */ var goog = {};`)
 - `null` or `undefined`
 - expressions consisting of literals (`const x = 'there are ' + 0 + ' cats.'`)
 - qualified names that represent types (`class Foo {} const FooAlias = Foo;`)
 - casts (`const x = /** @type {string} */ (fnCall());
 - qualified names with a declared type. (`const x = 0; const y = x;`)
 - new expressions of declared qualified names (`const n = new Number(0);`)

### Differences between inferred and declared types

There are three main distinctions between declared types and inferred types. The first is probably the most obvious: the compiler reports a warning if you assign the wrong type to a declared variable.

The other two distinctions stem from how the Closure Compiler implements type inference internally.

First, only names or properties with declared types are valid to reference in type annotations. This is because Closure Compiler resolves type annotations before doing any flow-sensitive type inference. This matters when, for example, aliasing types:

``` js
class Foo {}
const FooConstAlias = Foo;
let FooNonConstAlias = foo;
var /** !Foo */ f1;  // typed as a Foo.
var /** !FooConstAlias */ f2; // no warning, `f2` is typed as a Foo.
var /** !FooNonConstAlias */ f3;  // "bad type annotation" warning, because FooNonConstAlias does not have a declared type
```

Second, inferred properties on instance types (`this.x = 0;`) do not get typechecked well. This is more or less because instance properties references are nonlocal to where the type itself is declared. Here's an [example](https://closure-compiler-debugger.appspot.com/#input0%3D%252F**%2520%2540param%2520%257B!Person%257D%2520p%2520*%252F%250Afunction%2520analyze(p)%2520%257B%250A%2520%2520%252F%252F%2520See%2520if%2520Closure%2520Compiler%2520notices%2520that%2520you're%2520assigning%2520non-null%2520values%2520to%2520the%2520%2560null%2560%2520type.%250A%2520%2520const%2520%252F**%2520null%2520*%252F%2520name%2520%253D%2520p.name%253B%2520%252F%252F%2520warning%250A%2520%2520const%2520%252F**%2520null%2520*%252F%2520shoeSize%2520%253D%2520p.shoeSize%253B%2520%252F%252F%2520warning%250A%2520%2520const%2520%252F**%2520null%2520*%252F%2520age%2520%253D%2520p.age%253B%2520%252F%252F%2520no%2520warning%250A%2520%2520const%2520%252F**%2520null%2520*%252F%2520height%2520%253D%2520p.height%253B%2520%252F%252F%2520no%2520warning%250A%257D%250Aclass%2520Person%2520%257B%250A%2520%2520constructor(%252F**%2520string%2520*%252F%2520name%252C%2520%252F**%2520number%2520*%252F%2520height)%2520%257B%250A%2520%2520%2520%2520%252F**%2520%2540const%2520*%252F%250A%2520%2520%2520%2520this.name%2520%253D%2520name%253B%2520%2520%252F%252F%2520declared%250A%2520%2520%2520%2520%252F**%2520%2540type%2520%257B%253Fnumber%257D%2520*%252F%250A%2520%2520%2520%2520this.shoeSize%2520%253D%2520null%253B%2520%2520%252F%252F%2520declared%250A%2520%2520%2520%2520this.height%2520%253D%2520height%253B%2520%252F%252F%2520inferred%250A%2520%2520%2520%2520this.age%2520%253D%2520101%253B%2520%2520%2520%2520%252F%252F%2520inferred%250A%2520%2520%257D%250A%257D%2520%2520%250Aanalyze(new%2520Person('Bob%2520the%2520Builder'%252C%252061))%253B%26input1%26conformanceConfig%26externs%26refasterjs-template%26CHECK_TYPES%3Dtrue%26CLOSURE_PASS%3Dtrue%26PRESERVE_TYPE_ANNOTATIONS%3Dtrue%26PRETTY_PRINT%3Dtrue) of a function with a few potential type errors, only some of which are caught by Closure Compiler:

``` js
/** @param {!Person} p */
function analyze(p) {
  // See if Closure Compiler notices that you're assigning non-null values to the `null` type.
  const /** null */ name = p.name; // warning
  const /** null */ shoeSize = p.shoeSize; // warning
  const /** null */ age = p.age; // no warning
  const /** null */ height = p.height; // no warning
}
class Person {
  constructor(/** string */ name, /** number */ height) {
    /** @const */
    this.name = name;  // declared
    /** @type {?number} */
    this.shoeSize = null;  // declared
    this.height = height; // inferred
    this.age = 101;    // inferred
  }
}  
analyze(new Person('Bob the Builder', 61));
```

### Access controls

The above sections explained how JSDoc annotations relate to type declarations. What about [access control checks checks](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler#visibility-checks)?

A name or property without a declared type can still have restricted visibility. For example:

``` js
class Person {
  constructor() {
    /** @private */
    this.count = 0;
  }
}
```

The property `count` on `Person` does not have a declared type. However, the compiler will still enforce that `count` is not accessed outside of the above file.

This brings up a confusing point about Closure: some writes to properties are treated as a property _declaration_, but some are not. Additionally, properties that have a visibility declaration may not have a declared type.