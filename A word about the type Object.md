## Overview
`@type {Object}` may seem like the right type for a lot of JavaScript patterns, but in the Closure Compiler type system, Object is a very loose type. There are usually better alternatives.

## Namespaces

Namespaces are best typed as `@const` without a type annotation:

    /** @const */ var ns = {};
    /** @const */ ns.anInnerNamespace = {};

This tells the Closure Compiler to track the object as a unique, anonymous type. By contrast, annotations like `@const {Object}` and `@type {Object}` tell the compiler to treat it as any Object in the program (including any subtype of Object).

Note: `@const` indicates only that the name to which the value is assigned is never overwritten.  In particular, for a namespace, it does not indicate that the set properties of the object fix nor that the properties values are themselves immutable.

## Enumerations

Objects used to define a set of values should be typed with `@enum`. Even if you don't use the enum name as a type anywhere, the enum definition restricts the object type and allows strong typing of the referenced values.

    /** @enum {ValueType} */
    var enum = {
      KEY: value
    };

## Maps

Objects used as maps for runtime additions or lookups should restrict their key and value types using generics.

    /** @type {!Object<number, number>} */
    var map = {};
    
    /** @const {!Object<string, ValueType>} */
    var map = {
      'string key': value
    };

## Records

"Object" when used as a function parameter type when expecting a collection of properties, are often better expressed as a record type:

    /** @param {{key:value}} props */
    function f(props) {}

Where optional values should include "undefined":

    /** @param {{required:string, optional:(string|undefined)}} props */
    function f(props) {}

Record types tend to be verbose, and a typedef are often more practical:

    /** @typedef {{required:string, optional:(string|undefined)}} */
    var Params;

    /** @param {Params} props */
    function f(props) {}


Note that record types are non-nullable by default.

## Weak namespace type warnings

If you got here because you got a warning from the compiler about a weak namespace type and simply want to silence the warnings, you can use the type `Object<?,?>` which will tell the compiler that you are deliberately using a weak type.
