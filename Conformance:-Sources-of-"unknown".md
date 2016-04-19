# Improving type information: sources of “unknown” 

## Background

You got a conformance warning that the type of a property is of an unknown type in your
javascript, now what? This document attempts to list some common sources of
unknown properties and how to resolve them.

## Referencing properties defined on a subclass from a superclass type

Traditionally, the Closure Compiler has allowed subclass properties to be referenced
from super classes but such properties types are treated as “unknown” (as
generally there could be conflicting property definitions between subclasses and
all the subclasses may not be know when type checking is being done).

There are two things you should do:

1) declare the classes as @struct so the compiler warns about subclass property
references(ES6 classes and classes defined with goog.defineClass
behave as @struct classes by default)

2) change your code so that you know what subclass you are dealing with.

For the second, this may be a simple as templating the function where you are
getting the type. 

For example, for super class methods that return “this”, let the compiler know
it should return the :

```
/** 
 * @return {THIS}
 * @this {THIS}
 * @template THIS
 */
SomeClass.method = function() {
  ...
  return this;
}
```

Sometimes you may need to cast:

```
/** @type {SubClass} */(cls)

Or a runtime check:
goog.asserts.assertInstanceof(cls, SubClass)
```
For example:

```
/** @param {wiz.Controller} a */
function fn(ctlr) {
  // The type of ctlr needs to be tightened as #getValue
  // is not defined on wiz.Controller.
  return goog.asserts.assertInstanceof(ctlr, foo.bar.MyController).getValue();
}
```
## Using an Object as a map

Roughly, this pattern:

```
var x = {};
x[prop] = value;
use(x[prop]);
```
This results in a “unknown” for “x[prop]”. The Closure type system has a way of
annotating these kinds of objects:

```
/** @const {!Object&lt;string, ValueType>} */ var x = {};
```
NOTE: @const or @type can be used, but @const is preferred where possible.

NOTE: “ValueType” in this example, can be any valid type expression such as a
function or a union.

## Using a “map” Object for ad hoc properties

Roughly, this pattern:

```
/** @param {!Object&lt;string, ValueType>} x */
function f(x) {
  x.something = value;
  use(x.something);
}
```
This results in a “unknown” for “x.something”. “map” Object type parameters
only apply to computed properties ([] access)

## Unspecified Generic Types

Closure Compiler supports generic types through the use of the @template tag. Just
like in other languages, these generics have to be propagated from call sites
and parameter types. 

```
/**
 * @param {T} value
 * @template T
 */
var MyClass = function(value) {
  /** @const */
  this.value = value;
};
```
When this class is used as a type, the generic must be specified:

```
/**
 * @param {MyClass&lt;number>} obj
 */
function myFunction(obj) {
  goog.asserts.assert(obj.value instanceof number);
}
```
Failure to include the generics can cause unknown types to be propagated.  

## “Unknown” types warnings for well defined properties

If you see a warning like:

```
file.js:20  Violation: Reference to props that is typed as "unknown"
# The property "end" on type "goog.math.Range"
```

However, “end” is properly defined for goog.math.Range.

There are several variations that can cause this problem.

### The variable with the “known” type is undeclared

If the variable does not have an explicit declaration (an @type declaration)
but is set multiple times (and can’t be inferred as @const) and used with a
capturing function expression.


```
function method() {
  var x = foo();  // x is known
  if (y) {
    x = bar();  // x is known
  }
  function f() {
    use(x.prop);  // unknown prop warning here.
  }
}
```
SOLUTION 1: declare the type of x

SOLUTION 2: avoid setting x more than once

SOLUTION 3: introduce a temporary variable that is only set once

### The variable with the “known” type is declared

This is a known limitation of the current type inference.  The exact condition are:

  * the variable is declared in a method
  * the value is set in a function closure of within that method
  * the value is “tightened” during type inference (and <strong>if</strong> condition or an assert for example to remove null)
  * the value is then used

There are some some variations of this where the type appears to be known but
the type inference choose to ignore the type.

SOLUTION: add a type cast
