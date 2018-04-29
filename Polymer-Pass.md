# Polymer Pass for the Closure Compiler

## Overview

Polymer elements require transformations and custom type information to make them compatible with Closure-compiler.
The Polymer Pass recognizes elements and adds the needed type information and stub declarations so that the compiler
can effectively analyze and optimize them.

## Versions

The transformations performed vary by the version of Polymer used. The version is indicated by the `--polymer_version` flag.

~~Polymer 2 versions require new type inference `--new_type_inf`. The compiler's old type inference system does not
recognize the version 2 mixin constructs.~~

## Recognizing a Polymer Element

In either version 1 or 2, all calls to the `Polymer()` factory are properly recognized.

In version 2, you **must** annotate your class with `@polymer` if it does not directly extend
`Polymer.Element`. The compiler does not automatically recognize classes as having Polymer
symantics unless it either directly extends `Polymer.Element` or is annotated with `@polymer`.

```js
Polymer({}) // always recognized
class FooElement extends Polymer.Element {} // automatically recognized
/** @polymer */
class BarElement extends SomeOtherThing {} // recognized by the @polymer annotation
```



## Declared Property Typing

Properties should be annotated at their top-level definition, and not at Polymer's `type` field (which can only be one of String, Boolean, Number, Array or Object).

```js
class FooElement extends Polymer.Element {
  static get is() { return 'foo-element'; }
  static get properties() {
    return {
      /** @type {Array<number>} */
      bar: {
        type: Array,
        value: () => []
      },
      foo: Boolean // type automatically inferred
    };
  }
}
```

The compiler can infer the type of boolean, string and number typed declared properties without annotation.
The compiler will also infer `Object` and `Array` types, but these are usually not precise enough. It's
recommended to add generic type indicators for Objects and Arrays.

The Polymer Pass will add stub property definitions so that the compiler recognizes that these
properties are created on the class prototype.

## Observers and Computed Properties

Polymer observers and computed properties rely on a string reference to a method. The compiler 
will rename the methods and properties, and thereby break these references.

The compiler recognizes some intrinsic definitions to rename a string in the same manner as an
unquoted property.

```js
// Use reflection intrinsics to get a string renamed consistently
// with the `barChanged` property method
const barChangedName = goog.reflect.objectProperty('barChanged',
    /** @type {!FooElement} */ (/** @type {?} */ ({})));

class FooElement extends Polymer.Element {
  static get is() { return 'foo-element'; }
  static get properties() {
    return {
      bar: {
        type: String,
        observer: barChangedName
      }
    };
  }

  barChanged(bar) {}
}
```

Property reflection intrinsics:

 * [goog.reflect.objectProperty](https://google.github.io/closure-library/api/goog.reflect.html#objectProperty)
 * [goog.reflect.object](https://google.github.io/closure-library/api/goog.reflect.html#object)

 **Note:** It's not nesseccary to bring in all of closure-library to use these intrinsics.
 They are simply identity functions. Defining methods with these names in your code base
 is all that is required.

### Protecting Observer and Computed Property Methods from Dead Code Elimination
Property reflection intrinsics will not prevent a method from being eliminated as dead code.
The following snippet may be added for the compiler to recognize that such methods must not
be eliminated:

```js
/** @suppress {uselessCode} */
(() => {
  FooElement.prototype.barChanged; 
})();
```

### Advanced Mode Property Renaming

Argument names to computed property functions and complex observer methods must also be renamed
using the property reflection primitives.

```js
const fooElement = /** @type {!FooElement} */ (/** @type {?} */ ({}));
const barChangedName = goog.reflect.objectProperty('barChanged', fooElement);
const barName = goog.reflect.objectProperty('bar', fooElement);

class FooElement extends Polymer.Element {
  static get is() { return 'foo-element'; }
  static get properties() {
    return {
      bar: String
    };
  }

  static get observers() {
    return [
      `${barChangedName}(${barName})` // Both method and argument name require reflection
    ]
  }

  barChanged(bar) {}
}
```

## Class Mixins

Polymer 2 mixin functions dynamically create a class based off the provided super class argument.
The compiler requires that an interface be created to describe the class and the proper annotations
added.

```js
/** @interface */
function SomeMixinInterface() {}

/** @return {string} */
SomeMixinInterface.prototype.bar = function() {};

/**
 * @param {function(new:Polymer.Element)} Superclass
 * 
 * The return annotation should not be added to this function,
 * or if present should be the unknown types
 */
function addSomeMixin(Superclass)  {
  /**
   * @polymer
   * @implements {SomeMixinInterface}
   */
  class SomeMixin extends Superclass {
    /** @return {string} */
    bar() {}
  }

  return SomeMixin;
}

/**
 * @constructor
 * @extends {Polymer.Element}
 * @implements {SomeMixinInterface}
 */
const SomeMixinElement = addSomeMixin(Polymer.Element);

/** @polymer */
class MyMixedEleement extends SomeMixinElement {}
```

**WARNING** For use with ADVANCED mode and property renaming, the mixin interface must be complete.
Any properties (even private ones) provided by the mixed class must be represented in the interface or renaming
collisions could occur.

## Element Type Names for 1.x/Hybrid Call Syntax

The naming conventions for types of Polymer elements used by the compiler follow the following rules:

If there's an explicit LHS target of the Polymer call, that's used as the type.
```js
foo.Bar = Polymer({ is: 'foo-thing'...}); // Type is foo.Bar
var Foo = Polymer({ is: 'foo-thing'...}); // Type is Foo
```

Otherwise, the generated type is what you said - FooThingElement. This matches the convention for JS types of native HTML elements.

```js
Polymer({ is: 'foo-thing'...}); // Type is FooThingElement
```

## Class Annotations for 1.x/Hybrid Call Syntax

A class level annotation may be added to a 1.x/Hybrid element on the `factoryImpl` method.

```js
Polymer({
  is: 'my-foo',

  /** @implements {MyExternalInterface} */
  factoryImpl: function() {}
});
```

## 1.x/Hybrid Behaviors

Behaviors in Polymer are traditional JS mixins and are not well supported nor understood by the compiler. It
is highly advised to migrate to 2.0 style class mixins instead.

Behaviors must:

 * Be defined on a global namespace
 * Be an object literal or or array literal
 * Be annotated with `@polymerBehavior`

In addition, only very weak type checking is enabled by the compiler.

```js
const myNs = {};

/**
 * @const
 * @polymerBehavior
 */
myNs.FunBehavior = {
  properties: {
    isHappy: Boolean,

    /** @type {number} **/
    count: {
      type: Number,
      readOnly: true
    }
  },

  /** @param {string} funAmount */
  doSomethingFun: function(funAmount) { alert('Something fun!'); },
};

Polymer({
  is: 'x-custom',
  behaviors: [ myNs.FunBehavior ],
});
```

## Advanced Mode Property Renaming Implications

As polymer html templates contain references to JS code, any JS symbol referenced in
HTML must properly accounted for. Either:

 1. All referenced properties be quoted or exported
 2. A tool must be used to update the HTML templates after compilation

### Renaming Tools

 * [PolymerRenamer](https://github.com/PolymerLabs/PolymerRenamer) - not compatible with type-based optimizations
 * [Polymer-Rename](https://github.com/Banno/polymer-rename)


### Declared Property Renaming

The Polymer Pass enforces the ALL_UNQUOTED property renaming policy of the compiler. Properties should be
consistently quoted. If the declaration is quoted, all references to the property should also be quoted.

#### Version 2
Most declared properties are renamed. The compiler will consistently rename the returned properties object literal keys
with the class prototype properties.

Properties marked as `readOnly` or with `reflectToAttribute` are not renamed. `readOnly` properties have synthetic
setters generated for them which are based on their renamed version. Properties marked as `reflectToAttribute` are
typically used to style or locate an element.

#### Version 1
**All** declared properties are blocked from renaming. The compiler generates external interfaces to block renaming
of these properties.