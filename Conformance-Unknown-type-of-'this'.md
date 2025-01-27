# Unknown type of "this"

The type of “this” in JavaScript can become unknown to the Closure
Compiler for a few reasons (covered below). When this happens, the compiler is
no longer able to type check the code that references that object and limits the optimizations the compiler can perform on the code. However, there are a number of ways to prevent this from
happening depending on the situation of the code. You can also add a
conformance check to prevent it from coming back. See the BanUnknownThis conformance check

# Templatized Functions

If a function accepts another function as an argument (a callback) and then
calls the callback with a scope, then the accepting function must be
templatized to ensure that the JS Compiler has the correct type for “this”.

Ex:

<strong>WRONG</strong>

```
/**
 * Example function that is not properly templatized.
 * @param {function()} fn
 * @param {Object} scope
 */
function notTemplatized(fn, scope) {
  fn.call(scope);
}
```
When the “fn” is called, the JS Compiler will not know the correct type of the
“this” property when it is used within the callback.

<strong>RIGHT</strong>

```
/**
 * Example function that is properly templatized.
 * @param {function(this:THIS)} fn
 * @param {THIS} scope
 * @template THIS
 */
function templatized(fn, scope) {
  fn.call(scope);
}
```
In this case, the Closure Compiler does have the correct knowledge to infer the type
of “this”.

# Anonymous Functions

Like the templatized case above, the Closure Compiler can lose type information for
“this” when an anonymous function is used but it is not passed to another
function as a callback. In this case, the anonymous function can be used
elsewhere, such as immediately inline in the code.

<strong>WRONG</strong>


```
MyClass.prototype.someFunction = function() {
  function unknownThis() {
    this.someProperty = 5;
  }

  unknownThis.call(this);
};
```
When the unknownThis function is called, the Closure Compiler will not know the
correct type for “this” inside the function.

To fix this, the “@this” JS Doc annotation can be used:

<strong>RIGHT</strong>


```
MyClass.prototype.someFunction = function() {
  /** @this {MyClass} */
  function unknownThis() {
    this.someProperty = 5;
  }

  unknownThis.call(this);
};
```
# Missing @constructor on class

If a class is missing the @constructor annotation on the constructor, the JS
Compiler will treat any reference to “this” within the class as unknown.

<strong>WRONG</strong>


```
function MyClass() {
  // Unknown property
 this.someProperty = 5;
}
```
<strong>RIGHT</strong>


```
/**
 * @constructor
 */
function MyClass() {
  this.someProperty = 5;
}
```
# Missing @extends annotation

If a class extends another class (either using goog.inherits or assigning the
prototype property), the constructor should have an @extends annotation in
order to make sure that the Closure Compiler understands the code.

<strong>WRONG</strong>


```
/** @constructor */
function BaseClass() {}

/** @constructor */
function SubClass() {}
SubClass.prototype = new BaseClass();
SubClass.prototype.constructor = SubClass;
SubClass.superclass = BaseClass.prototype;
```
This code should make sure to have @extends on the constructor (and use
goog.inherits).

<strong>RIGHT</strong>


```
/** @constructor */
function BaseClass() {}

/**
 * @extends {BaseClass}
 * @constructor
 */
function SubClass() {}
goog.inherits(SubClass, BaseClass);
```
# Modifying a class prototype without goog.requiring the class

It is generally discouraged to modify the prototype of a class that is defined
in a different file. If so,
that code should make sure to goog.require the class that it is modifying.

<strong>WRONG</strong>

```
ClassInOtherFile.prototype.someFunction = function() {
  this.someProperty = 5;
};
```
<strong>RIGHT</strong>

```
goog.require(‘ClassInOtherFile’);

ClassInOtherFile.prototype.someFunction = function() {
  this.someProperty = 5;
};
```
# Forward Declarations

The Closure Compiler does not have the class or
method signatures for forward declarations, so if a file only refers to a
type without including that file in the compilation unit, this could cause the
Closure Compiler to report the “this” is unknown.

# Templated @this declarations

If a method or function declares a @this with a template type, such as:


```
/** 
 * @this {T}
 * @return {T}
 * @template T
 */
```
The method will have an unknown “this” within the function body.  This can be
fixed with the use one of three methods:


```
  goog.asserts.assertInstanceof(this, Foo);

  var self = /** @type {Foo} */ (this);

  if (this instanceof Foo) {
    ...
  }
```
---

