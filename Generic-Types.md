# Generic Types

Much like Java, the Closure Compiler supports generic types, functions, and methods. Generics operate on objects of various types while preserving compile-time type safety.

You can use generics to implement generalized collections that hold references to objects of a particular type, and generalized algorithms that operate over objects of a particular type.

### Declaring a Generic Type

A type can be made generic by adding a `@template` annotation to the type's constructor (for classes) or interface declaration (for interfaces). For example:

```javascript
/**
 * @constructor
 * @template T
 */
Foo = function() { ... };
```

The annotation `@template T` indicates that Foo is a generic type with one template type, `T`. The template type `T` can be used as a type within the scope of the definition of Foo. For example:

```javascript
/** @return {T} */
Foo.prototype.get = function() { ... };

/** @param {T} t */
Foo.prototype.set = function(t) { ... };
```

The method get will return an object of type `T`, and the method set will only accept objects of type `T`.

### Instantiating a Generic Type

Reusing the example above, a templated instance of Foo can be created in several ways:

```javascript
/** @type {!Foo<string>} */ var foo = new Foo();

var foo = /** @type {!Foo<string>} */ (new Foo());
```

Both of the above constructor statements create a Foo instance whose template type `T` is string. The compiler will enforce that calls to foo's methods, and accesses to foo's properties, respect the templated type. For example:


```javascript
foo.set("hello");  // OK.
foo.set(3);        // Error - expected a string, found a number.
var x = foo.get(); // x is a string.
```

Instances can also be implicitly typed by their constructor arguments. Consider a different generic type, Bar:

```javascript
/**
 * @param {T} t
 * @constructor
 * @template T
 */
Bar = function(t) { ... };

var bar = new Bar("hello"); // bar is a Bar<string>
```

The type of the argument to the Bar constructor is inferred as string, and as a result, the created instance bar is inferred as `Bar<string>`.

### Multiple Template Types

A generic can have any number of template types. The following map class has two template types:

```javascript
/**
 * @constructor
 * @template Key, Val
 */
MyMap = function() { ... };
```

All template types for a generic type must be specified in the same @template annotation, as a comma-separated list. The order of the template type names is important, since templated type annotations will use the ordering to pair template types with the values. For example:

```javascript
/** @type {MyMap<string, number>} */ var map; // Key = string, Val = number.
```

### Invariance of Generic Types

The Closure Compiler enforces invariant generic typing. This means that if a context expects a type `Foo<X>`, you cannot pass a type `Foo<Y>` when X and Y are different types, even if one is a subtype of the other. For example:

```javascript
/**
 * @constructor
 */
X = function() { ... };

/**
 * @extends {X}
 * @constructor
 */
Y = function() { ... };

/** @type {Foo<X>} */ var fooX;
/** @type {Foo<Y>} */ var fooY;

fooX = fooY; // Error
fooY = fooX; // Error

/** @param {Foo<Y>} fooY */
takesFooY = function(fooY) { ... };

takesFooY(fooY); // OK.
takesFooY(fooX); // Error
```

### Inheritance of Generic Types

Generic types can be inherited, and their template types can either be fixed or propagated to the inheriting type. Here is an example of an inheriting type fixing the template type of its supertype:

```javascript
/**
 * @constructor
 * @template T
 */
A = function() { ... };

/** @param {T} t */
A.prototype.method = function(t) { ... };

/**
 * @constructor
 * @extends {A<string>}
 */
B = function() { ... };
```

By extending `A<string>`, `B` will have a method method that takes a parameter of type string.

Here is an example of an inheriting type propagating the template type of its supertype:


```javascript
/**
 * @constructor
 * @template U
 * @extends {A<U>}
 */
C = function() { ... };
```

By extending `A<U>`, templated instances of C will have a method method that takes a parameter of the template type `U`.

Interfaces can be implemented and extended in a similar fashion, but a single type cannot implement the same interface multiple times with different template types. For example:

```javascript
/**
 * @interface
 * @template T
 */
Foo = function() {};

/** @return {T} */
Foo.prototype.get = function() {};

/**
 * @constructor
 * @implements {Foo<string>}
 * @implements {Foo<number>}
 */
FooImpl = function() { ... }; // Error - implements the same interface twice
Generic Functions and Methods
```

Similar to generic types, functions and methods can be made generic by adding a `@template` annotation to their definition. For example:

```javascript
/**
 * @param {T} a
 * @return {T}
 * @template T
 */
identity = function(a) { return a; };

/** @type {string} */ var msg = identity("hello") + identity("world"); // OK
/** @type {number} */ var sum = identity(2) + identity(2); // OK
/** @type {number} */ var sum = identity(2) + identity("2"); // Type mismatch
```