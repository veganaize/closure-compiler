# Generic Types

Much like Java, the Closure Compiler supports generic types, functions, and methods. Generics operate on objects of various types while preserving compile-time type safety.

You can use generics to implement generalized collections that hold references to objects of a particular type, and generalized algorithms that operate over objects of a particular type.

### Declaring a Generic Type

A class or interface can be made generic by adding a `@template` annotation to the class's JSDoc. For example:

```javascript
/** @template T */
class Foo {}
```

Pre-ES6 style classes and interfaces are made generic by adding a `@template` annotation to the constructor JSDoc. For example:

```javascript
/**
 * @constructor
 * @template T
 */
var Foo = function() { ... };
```

The annotation `@template T` indicates that Foo is a generic type with one template type, `T`. The template type `T` can be used as a type within the scope of the definition of Foo. For example:

```javascript
/** @template T */
class Foo {
  /** @return {T} */
  get() { ... };

  /** @param {T} t */
  set(t) { ... };
}
```

The method `get` will return an object of type `T`, and the method `set` will only accept objects of type `T`.

Pre-ES6, this would look like:

```javascript
/** @return {T} */
Foo.prototype.get = function() { ... };
/** @param {T} t */
Foo.prototype.set = function(t) { ... };
```

### Declaring a Bounded Generic Type

A template can be declared with an upper bound, specifying that any type the template assumes must be a subtype of the annotated bound. This is similarly annotated using the `@template` tag, but placing the type expression bound in curly braces between the template name. For example:

```javascript
/** @template {!Element} T
class Foo {}
```

Values of generic types that share the same bound are still unassignable to each other. Because the template can represent any subtype of the bound, it is unknown whether one generic is a subtype of the other or not. For this reason, generic types that share the same bound are invariant. For example:

```javascript
/**
 * @param {T} x
 * @param {S} y
 * @template {number|string} T
 * @template {number|string} S
 */
function foo(x,y) { x=y; }    // Error, y=x would error as well
```

Generic types can be bounded by type expressions containing other generic types declared in the same scope. When a generic type is directly bounded by another generic type, this makes the first generic assignable to the second. For example:

```javascript
/**
 * @param {T} x
 * @param {S} y
 * @template {number|string} T
 * @template {T} S
 */
function foo(x,y) { x=y; }    // No error
```

This scoping allows generics to be recursively defined. However, direct cycles of template type bounds are forbidden. For example: 

```javascript
/**
 * @template {S} T // Error
 * @template {T} S // Error
 */
```

### Instantiating a Generic Type

Reusing the example above, a templated instance of Foo can be created in several ways:

```javascript
/** @type {!Foo<string>} */ const foo = new Foo();

const foo = /** @type {!Foo<string>} */ (new Foo());
```

Both of the above constructor statements create a Foo instance whose template type `T` is string. If `T` is a bounded template type, a check is conducted to make sure the templatized type respects the type bound given to `T`. The compiler will enforce that calls to foo's methods, and accesses to foo's properties, respect the templatized type. For example:

```javascript
foo.set("hello");    // OK.
foo.set(3);          // Error - expected a string, found a number.
const x = foo.get(); // x is a string.
```

Instances can also be implicitly typed by their constructor arguments. Consider a different generic type, Bar:

```javascript
/** @template T */
class Bar {
  /** @param {T} t */
  constructor(t) { ... }
}

const bar = new Bar("hello"); // bar is a Bar<string>
```

The type of the argument to the Bar constructor is inferred as string, and as a result, the created instance bar is inferred as `Bar<string>`.

### Multiple Template Types

A generic can have any number of template types. The following map class has two template types:

```javascript
/** @template Key, Val */
class MyMap { ... }
```

All template types for a generic type must be specified in the same @template annotation, as a comma-separated list. The order of the template type names is important, since templated type annotations will use the ordering to pair template types with the values. For example:

```javascript
/** @type {!MyMap<string, number>} */ let map; // Key = string, Val = number.
```


### Multiple Bounded Template Types

Multiple bounded generics cannot be declared on the same line. For the sake of clarity, if multiple templates share the same type bound they must be declared on separate lines. This eliminates any ambiguity as to which template a bound is meant to apply to.

```javascript
/**
 * @template {Foo} T     // Okay
 * @template {Foo} S     // Okay
 * @template {Foo} U,V   // NOT okay
 */
```

### Invariance of Generic Types

The Closure Compiler enforces invariant generic typing. This means that if a context expects a type `Foo<X>`, you cannot pass a type `Foo<Y>` when X and Y are different types, even if one is a subtype of the other. For example:

```javascript
class X {}
class Y extends X {}

/** @type {!Foo<X>} */ let fooX;
/** @type {!Foo<Y>} */ let fooY;

fooX = fooY; // Error
fooY = fooX; // Error

/** @param {!Foo<Y>} fooY */
const takesFooY = function(fooY) { ... };

takesFooY(fooY); // OK.
takesFooY(fooX); // Error
```

### Inheritance of Generic Types

Generic types can be inherited, and their template types can either be fixed or propagated to the inheriting type. Here is an example of an inheriting type fixing the template type of its supertype:

```javascript
/** @template T */
class A {
  /** @param {T} t */
  method(t) { ... }
}

/** @extends {A<string>} */
class B extends A { ... }
```

By extending `A<string>`, `B` will have a method that takes a parameter of type string.

Note that this example uses both the `extends` keyword and `@extends` in JSDoc. This is only necessary because the superclass is templatized.

Here is an example of an inheriting type propagating the template type of its supertype:


```javascript
/**
 * @template U
 * @extends {A<U>}
 */
class C extends A { ... }
```

By extending `A<U>`, templated instances of C will have a method that takes a parameter of the template type `U`.

Interfaces can be implemented and extended in a similar fashion, but a single type cannot implement the same interface multiple times with different template types. For example:

```javascript
/**
 * @interface
 * @template T
 */
class Foo {
  /** @return {T} */
  get() {}
}

/**
 * @implements {Foo<string>}
 * @implements {Foo<number>}
 */
class FooImpl { ... }; // Error - implements the same interface twice
```

### Generic Functions and Methods

Similar to generic types, functions and methods can be made generic by adding a `@template` annotation to their definition. For example:

```javascript
/**
 * @param {T} a
 * @return {T}
 * @template T
 */
const identity = (a) => a;

/** @type {string} */ const msg = identity("hello") + identity("world"); // OK
/** @type {number} */ const sum1 = identity(2) + identity(2);            // OK
/** @type {number} */ const sum2 = identity(2) + identity("2");          // Type mismatch
```

### Limitations

You can reference generic type parameters inside a function body. However, with the exception of instance variables declared in a constructor, they always produce the unknown type if unbounded. For example, this code produces no error:

```javascript
/**
 * @param {T} t
 * @template T
 */
function f(t) {
  const /** null */ n = t;  // No warning
}
```

However, similar code using a bounded generic template, does produce an error.
```javascript
/**
 * @param {T} t
 * @template {!Object} T
 */
function f(t) {
  const /** null */ n = t;  // Warning!
}
```