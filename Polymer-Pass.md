# Polymer Pass for the Closure Compiler

*__Status:__ Draft*&nbsp;&nbsp;&nbsp;__|__&nbsp;&nbsp;&nbsp;*__Author:__ @jklein24*&nbsp;&nbsp;&nbsp;__|__&nbsp;&nbsp;&nbsp;&nbsp;*__Last Updated:__ 2015/04/02*


## Objective

Write a new JSCompiler pass for calls to the Polymer({}) function for custom element registration. This pass will provide better type information to the compiler and allow for some basic compile-time checks on Polymer custom elements.

## Overview

*TODO (jklein24): Give a high level overview of transformations.*

## Using with property renaming

*TODO (jklein24): Describe usage with [PolymerRenamer](https://github.com/PolymerLabs/PolymerRenamer).*

## Detailed Design

There are several features of the Polymer element registration descriptor which have implications for compilation. In writing this document, I have used the [Polymer 0.8 primer](https://github.com/Polymer/polymer/blob/0.8-preview/PRIMER.md) and [polymer-project.org](https://www.polymer-project.org/1.0/docs/devguide/feature-overview.html) as the canonical reference for these features and will note the most relevant ones and how they are addressed by this design.

### Element Type Names

The naming conventions for types of Polymer elements used by the compiler follow the following rules:

1. If there's an explicit LHS target of the Polymer call, that's used as the type.
```
    1. foo.Bar = Polymer({ is: 'foo-thing'...}); // Type is foo.Bar

    2. var Foo = Polymer({ is: 'foo-thing'...}); // Type is Foo
```
2. Otherwise, the generated type is what you said - FooThingElement. This matches the convention for JS types of native HTML elements.
```
    3. Polymer({ is: 'foo-thing'...}); // Type is FooThingElement
```
### [Bespoke Constructor](https://github.com/Polymer/polymer/blob/0.8-preview/PRIMER.md#bespoke-constructor-support)

Polymer allows for the following syntax to construct custom elements:

Original Code:
```
Polymer({
  is: ‘my-foo’,

  /**
   * @param {string} bar
   */
  factoryImpl: function(bar) {
    this.bar = bar;
  },
});
```
Generated Code:
```
/**
 * @param {string} bar
 * @constructor @extends {PolymerElement}
 */
var MyFooElement = function(bar) {
  this.bar = bar;
};
Polymer(/** @lends {XCustomElement.prototype} */ {
  is: ‘my-foo’,
  factoryImpl: function(bar) {
    this.bar = bar;
  },
});
```
### [Native Element Extension](https://github.com/Polymer/polymer/blob/0.8-preview/PRIMER.md#native-html-element-extension)

Polymer elements cannot extend other custom elements, but they can extend native HTML elements like input, select, etc. If a custom element, foo-bar, does not extend any native HTML element, its prototype chain is "foo-bar  ->  PolymerElement  ->  HTMLElement". For an element which extends an input, the prototype chain should be “foo-bar -> PolymerElement -> HTMLInputElement”. However, conditionally inheriting different elements in the externs is not really possible. The solution is to make new copies of the PolymerElement externs which extend the right native element.

Original Externs:
```
/** @constructor @extends {HTMLElement} */
var PolymerElement = function() {};

/** @type {Object.<string, !HTMLElement>} */
PolymerElement.prototype.$;
PolymerElement.prototype.created = function() {};

etc….
```

Original Code:
```
MyInput = Polymer({
  is: 'my-input',
  extends: 'input',
  created: function() {
    this.style.border = '1px solid red';
  }
});
```

Generated Externs:
```
/** @constructor @extends {HTMLElement} */
var PolymerElement = function() {};

/** @type {Object.<string, !HTMLElement>} */
PolymerElement.prototype.$;
PolymerElement.prototype.created = function() {};

etc….

/** @constructor @extends {HTMLInputElement} */
var PolymerInputElement = function() {};
/** @type {Object.<string, !HTMLElement>} */
PolymerInputElement.prototype.$;
PolymerInputElement.prototype.created = function() {};

etc….
```

Generated Code:
```
/** @constructor @extends {PolymerInputElement} */
MyInput = function() {};
MyInput = Polymer(/** @lends {XCustomElement.prototype} */ {
  is: 'my-input',
  extends: 'input',
  created: function() {
    this.style.border = '1px solid red';
  }
});
```

### [Properties Block](https://github.com/Polymer/polymer/blob/0.8-preview/PRIMER.md#configuring-properties)

For properties blocks, keys must be pulled onto the prototype of the Element. The type is determined first by any existing JSDocInfo on the property. If no type is specified, the type will be inferred from the value in the properties block. Original docs in the object literal are removed.

Original Code:
```
Polymer({
  is: 'x-custom',
  properties: {
    /**
     * The user of the thing.
     * @type {!foo.bar.User}
     */
    user: Object,
    isHappy: Boolean,
    count: {
      type: Number,
      notify: true
    }
  },
});
```

Generated code:
```
/** @constructor @extends {PolymerElement} */
var XCustomElement = function() {};
/** @type {!foo.bar.User} */
XCustomElement.prototype.user;
/** @type {boolean} */
XCustomElement.prototype.isHappy;
/** @type {number} */
XCustomElement.prototype.count;
Polymer(/** @lends {XCustomElement.prototype} */{
  is: 'x-custom',
  properties: {
    user: Object,
    isHappy: Boolean,
    count: {
      type: Number,
      notify: true
    }
  },
});
```

### [this.$.innerElement](https://www.polymer-project.org/1.0/docs/devguide/local-dom.html#node-finding)

Polymer allows for automatic local node finding using this.$.elementID. However, there is no great way to annotate this.$ in the Polymer externs because the PolymerPass doesn’t know the list of ids or the type of elements they refer to at compile time. The pass also needs to avoid renaming of these ids. Therefore, the easiest thing to do is to make the pass automatically switch from this.$.foo to this.$[‘foo’].

Original Code:
```
/** @constructor */
var SomeType = function() {};
SomeType.prototype.toggle = function() {};
SomeType.prototype.switch = function() {};
SomeType.prototype.touch = function() {};

var X = Polymer({
  is: 'x-element',
  sayHi: function() {
    this.$.checkbox.toggle();
  },
  /** @override */
  created: function() {
    this.sayHi();
    this.$.radioButton.switch();
  },
  /**
   * @param {string} name
   * @private
   */
  sayHelloTo_: function(name) {
    this.$.otherThing.touch();
  },
})
```

Generated code:
```
/** @constructor */
var SomeType = function() {};
SomeType.prototype.toggle = function() {};
SomeType.prototype.switch = function() {};
SomeType.prototype.touch = function() {};
/** @constructor @extends {PolymerElement} @implements {PolymerXInterface} */
var X = function() {};
X = Polymer(/** @lends {X.prototype} */ {
  is: 'x-element',

  /** @this {X} */
  sayHi: function() {
    this.$['checkbox'].toggle();
  },

  /** @override @this {X} */
  created: function() {
    this.sayHi();
    this.$['radioButton'].switch();
  },

  /**
   * @param {string} name
   * @private
   * @this {X}
   */
  sayHelloTo_: function(name) {
    this.$['otherThing'].touch();
  },
})
```

### [Readonly _set<Prop> functions](https://www.polymer-project.org/0.8/docs/devguide/properties.html#read-only)

These setter functions are generated by polymer at runtime for readOnly properties. They are private and cannot be renamed at all. To completely avoid renaming, the pass generates an interface for the element at compile time and adds any required _set* functions to that interface. It then puts the whole interface into externs.

Original Code:
```
Polymer({
  is: 'x-custom',
  properties: {
    isHappy: Boolean,
    count: {
      type: Number,
      readOnly: true
    }
  },
});
```

Generated Externs:
```
/** @interface */
var PolymerXCustomElementInterface = function() {};
/** @param {number} count **/
PolymerXCustomElementInterface.prototype._setCount;
```

Generated code:
```
/**
 * @constructor
 * @implements {PolymerXCustomElementInterface}
 * @extends {PolymerElement}
 */
var XCustomElement = function() {};
/** @type {boolean} */
XCustomElement.prototype.isHappy;
/** @type {number} */
XCustomElement.prototype.count;
/** @override */
XCustomElement.prototype._setCount;
Polymer(/** @lends {XCustomElement.prototype} */{
  is: 'x-custom',
  properties: {
    isHappy: Boolean,
    count: {
      type: Number,
      readOnly: true,
      notify: true
    }
  },
});
```

### [Behaviors](https://github.com/Polymer/polymer/blob/0.8-preview/PRIMER.md#behaviors)

For now, the goal is to simply avoid compiler errors with behaviors. Type checking won’t be quite as strong as possible because we don’t have a good solution yet to determine the "this" scope inside lifecycle functions for behaviors. The current implementation simply copies all behavior properties and non-lifecycle functions over to the prototype of the element definition. Array behaviors are parsed recursively. Behaviors must be fully qualified names in the global scope.

In order to avoid type errors for behavior definitions, type annotations on properties are stripped and checkTypes suppressions are added to every function in the definition. Note that the types will still be checked for any behavior which is actually used by an element because the functions will be copied over to the element’s prototype before adding any suppressions. Once caveat to this is that if a behavior is declared in non-global scope (like an iife), it is not possible to copy the entire scope so that variables inside behavior functions are valid in the context of the Element using this behavior. Therefore, for these behaviors, only a function stub is created and types cannot be checked. The team is still exploring better solutions to this problem.

Original Code:
```
/** @polymerBehavior */
var FunBehavior = {
  properties: {
    isHappy: Boolean,

    /** @type {number} **/
    count: {
      type: Number,
      readOnly: true
    }
  },

  /** @param {string} funAmount */",
  doSomethingFun: function(funAmount) { alert('Something fun!'); },
};

var A = Polymer({
  is: 'x-custom',
  behaviors: [ FunBehavior ],
});
```

Generated Externs:
```
/** @interface */
var PolymerAInterface = function() {};
/** @param {number} count **/
PolymerAInterface.prototype._setCount;
```

Generated code:
```
/** @polymerBehavior @nocollapse */
var FunBehavior = {
  properties: {
    isHappy: Boolean,
    count: {
      type: Number,
      readOnly: true
    },
  },

  /** @suppress {checkTypes} */",
  doSomethingFun: function(funAmount) { alert('Something fun!'); },
};

/**
 * @constructor
 * @implements {PolymerAInterface}
 * @extends {PolymerElement}
 */
var A = function() {};
/** @type {boolean} */
A.prototype.isHappy;
/** @type {number} */
A.prototype.count;
/** @override */
A.prototype._setCount;
A = Polymer({
  is: 'x-custom',
  behaviors: [ FunBehavior ],
});
```