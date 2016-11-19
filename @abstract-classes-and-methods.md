# Introduction
Historically, Closure Library has provided limited support for abstract methods via `goog.abstractMethod`, which throws an error at runtime if an abstract prototype method is called. However, because Closure Compiler did not have any notion of abstract classes and methods, it could not perform compile-time checks to enforce that abstract classes were not instantiated or abstract methods were not called. Supporting the `@abstract` JsDoc and using that to aid compile-time checks on the semantics of abstract classes and methods has been a feature request to Closure Compiler for a long time. This feature is also present in some other JavaScript optional type systems, such as TypeScript. Recently, Closure Compiler has added support for `@abstract` on classes and methods at the type system level. For new JavaScript code, the `@abstract` JsDoc should be used in favor of `goog.abstractMethod`.

# Examples
* An abstract ES6 class
```js
/** @abstract */
class Foo {
  constructor() {}
  /** @abstract */ bar() {}
}
```
* An abstract Closure style class 
```js
/** @abstract @constructor */
var Foo = function() {};
/** @abstract */   
Foo.prototype.bar = function() {};
```
# Rules
* Abstract classes cannot be instantiated with `new`.
```js
/** @abstract */
class Foo {}
let f = new Foo; // WARNING - cannot instantiate abstract class
```
* Abstract methods cannot have implementations.
```js
/** @abstract */
class Foo {
  /** @abstract */
  bar() {
    console.log('bar');
    // WARNING - Misplaced @abstract annotation. function with a 
    // non-empty body cannot be abstract
  }
}
```
* If a method is marked abstract, the class it resides in needs to be abstract.
```js
class Foo {
  /** @abstract */ bar() {}
  // WARNING - Abstract methods can only appear in abstract classes.
  // Please declare the class as @abstract
}
```
* Concrete subclasses of an abstract class must implement all the abstract methods.
```js
class Foo {
  /** @abstract */ bar() {}
}
class Bar extends Foo {
  // WARNING - property bar on abstract class Foo is not implemented
  // by type Bar
}
```
* Currently, abstract classes that implement interfaces must still declare the methods on those interfaces.
```js
/** @interface */
class I {
  foo() {}
}
/** @abstract @implements {I} */
class Foo {
  // WARNING - property foo on interface I is not implemented by type 
  // Foo
}
```
* `@private` and `@abstract` are incompatible.
```js
/** @abstract */
class Foo {
  /** @private @abstract */ bar() {}
  // WARNING - Bad type annotation. type annotation incompatible with 
  // other annotations
}
```
* `@final` and `@abstract` are incompatible.
```js
/** @abstract */
class Foo {
  /** @final @abstract */ bar() {}
  // WARNING - Bad type annotation. type annotation incompatible with 
  // other annotations
}
```
* Static methods cannot be `@abstract`.
```js
/** @abstract */
class Foo {}
Foo.bar = function() {};
// WARNING - Misplaced @abstract annotation. only functions or
// non-static methods can be abstract
```
* Abstract methods cannot be called via call or apply. However, constructors for abstract classes are allowed to be called via call or apply. Note that the constructor method in an ES6 class is not allowed to be marked abstract.
```js
/** @abstract */
class Foo {}
/** @abstract */ Foo.prototype.bar = function() {};
class Bar extends Foo {}

Foo.prototype.bar.call(new Bar);
// WARNING - Abstract method Foo.prototype.bar cannot be called

Foo.call(new Bar); // Allowed

/** @abstract */
class Baz {
  /** @abstract */ constructor() {}
  // WARNING - Misplaced @abstract annotation. constructors cannot be 
  // abstract
}
```
* Abstract methods cannot be called within a class member function body via `super.foo()` where `foo` is an abstract method belonging to the parent class. However, they may be called within a class member function body via `this.foo()` where `foo` is an abstract method belonging to that class.
```js
/** @abstract */
class Foo {
  /** @abstract */ foo() {}
  bar() {
    this.foo(); // Allowed
  }
}
class Bar extends Foo {
  /** @override */
  foo() {
    super.foo();
    // WARNING - Abstract method Foo.prototype.foo cannot be called
  }
  /** @override */ bar() {}
}
```