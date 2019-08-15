As of the v20150729 Closure Compiler release, Closure Compiler has structural interfaces and covariant record types. Here's an overview of what changes that means for users of the Closure Compiler type system.

## Nominal Interfaces

The oldest style of interface in the Closure Compiler is the nominal interface. For example, for:
````javascript
/** @interface */
function PNumI() {};
/** @type {number} */
PNumI.prototype.p;

/** @constructor */
function Foo() {};
/** @type {number} */
Foo.prototype.p = 5;
````
since Foo doesn't explicitly `@implement` PNumI, it isn't considered a subtype, even though it has all of the required properties for implementations of PNumI.
````
WARNING - initializing variable
found   : Foo
required: PNumI
var /** !PNumI */ x = new Foo;
                     ^
````

## Structural interfaces

With structural interfaces (declared with the `@record` annotation), you don't need an explicit `@implements` tag to get subtyping. In the following example:

````javascript
/** @record */
function PNumI() {};
/** @type {number} */
PNumI.prototype.p;

/** @constructor */
function Foo() {};
/** @type {number} */
Foo.prototype.p = 5;
````

`Foo` would be considered to match `PNumI`. Object literal types may also match structural interfaces:

````javascript
/** @record */
function PNum() {};
/** @type {number} */
PNum.prototype.p;

var /** !PNum */ x = new Foo;  // OK with structural interfaces
var /** !PNum */ y = { p: 5 }; // OK with structural interfaces
````

### Use of `@interface` is preferred to `@record` when possible

Defining a new `@record` can have the side-effect that one or more classes far away in the program code can now be considered to be a subtype of this new record. On the flip side, a new or modified class could be adding a new implementation of an `@record` type that is far away and unknown to the reader or writer of the code. This action-at-a-distance does not occur with `@interface` types.

Why does this matter? If you're using type-based optimizations (which are included in ADVANCED_OPTIMIZATIONS), using `@record` makes it harder for the compiler to optimize your code.

When you use `@record`, the compiler has to assume that any arbitrary class, `C`,  that happens to implement the same properties as some record, `R`, could be intended to implement it, unless it can prove that no instance of C is ever used in a place where an `R` is expected. Proving this is often very difficult or impossible. The result is often that `C` has to stay in the output JS as long anything using an `R` is there. This can lead to a significant increase in output code size.

## Record Types

The Closure Compiler types object literals in JavaScript as 'record types'. You can also explicitly create a record type in a type annotation with an object-literal like syntax. For example:

``` js
/**
 * @param {{size: number}} opts */
 */
function f(opts) {}
f({size: 42});
```

Record types may have optional properties. Declaring a property as a type union with `|undefined` makes it optional:

``` js
/**
 * @param {{height: number|undefined, width: number}} opts */
 */
function f(opts) {}
f({height: 24, width: 42}); // Fine
f({width: 42});  // Fine, since height is optional
f({});  // Not okay, since width is required.
```

### Records vs. Structural Interfaces

Since record types are already matched structurally, they are similar to structural interfaces. 
Unlike with traditional records, though, structural interfaces have names, and can be used
with `@implements`, `@extends`, etc.

Also, since structural types have a name, they can
be used recursively, as in:
````javascript
/** @record */
function Recursive(){};
/** @type {?Recursive} */
Recursive.prototype.next;
````

Also note that structural interfaces are declared through the `@record` annotation, but are not record types. Record types do not need an `@interface` or `@record` annotation.

### Covariant Records

In order to align records better with structural interfaces, release v20150729  converted record types from being invariant to being covariant.  Thus, programs like the following are now allowed by Closure Compiler:
``` javascript
function f(/** { p : number } */ x) {
  var /** { p : (number|string) } */ y = x;
}
```
