As of the v20150729 Closure Compiler release, we've been working toward having structural interfaces and covariant record types. Here's an overview of what changes that means for users of the Closure Compiler type system.

## Structural Interfaces

Currently, Closure Compiler has nominal interfaces. For example, for:
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

With structural interfaces (declared with the `@record` annotation), Foo would be considered to match. Since structural interfaces are very similar to the existing record types, records also match: 

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

Why does this matter? When you use `@record` the compiler has to assume that any arbitrary class, C,  that happens to implement the same properties as some record, R, could be intended to implement it, unless it can prove that no instance of C is ever used in a place where an R is expected. Proving this is often very difficult or impossible. The result is often that C has to stay in the output JS as long anything using an R is there. This can lead to a significant increase in output code size.

### Records vs. Structural Interfaces

Since record types are already matched structurally, they are similar to structural interfaces. 
Note that unlike with traditional records, though, structural interfaces have names, and can be used
with `@implements`, `@extends`, etc.

Also, since structural types have a name, they can
be used recursively, as in:
````javascript
/** @record */
function Recursive(){};
/** @type {?Recursive} */
Recursive.prototype.next;
````

### Covariant Records

In order to align records better with structural interfaces, we have also converted record types from being invariant to being covariant.  Thus, programs like the following are now allowed by Closure Compiler:
````javascript
function f(/** { p : number } */ x) {
  var /** { p : (number|string) } */ y = x;
}
````
