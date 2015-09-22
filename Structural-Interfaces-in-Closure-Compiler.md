As of the r20150729 Closure Compiler release, we've been working toward having structural interfaces and covariant record types. Here's an overview of what changes that means for users of the Closure Compiler type system.

## Structural Interfaces

Currently, Closure Compiler has nominal interfaces. For example, for:
````javascript
/** @interface */
function PNum() {};
/** @type {number} */
PNum.prototype.p;

/** @constructor */
function Foo() {};
/** @type {number} */
Foo.prototype.p = 5;
````
since Foo doesn't explicitly @implement PNum, it isn't considered a subtype, even though it has all of the required properties for implementations of PNum.
````
WARNING - initializing variable
found   : Foo
required: PNum
var /** !PNum */ x = new Foo;
                     ^
````

With structural interfaces, Foo would be considered to match PNum. Since structural interfaces are very similar to the existing record types, it also makes sense to let records match 

````javascript
var /** !PNum */ x = new Foo;  // OK with structural interfaces
var /** !PNum */ y = { p: 5 }; // OK with structural interfaces
````

For now, we use the `@record` annotation to denote structural interfaces, but we hope to convert all interfaces to structural since it's simpler to have only one style of interfaces.

### Records vs. Structural Interfaces

Since record types are already matched structurally, they are similar to structural interfaces. 
Note that unlike with traditional records, though, structural interfaces have names, and can be used
with @implements, @extends, etc.

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
