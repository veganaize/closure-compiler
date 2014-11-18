## Overview
`@type {Object}` seems like the right type for a lot of JavaScript patterns.  But in the Closure Compiler type system, Object is a very loose type. There are usually better alternatives.

## Namespaces

Namespaces are best typed as `@const` without a type annotation:

    /** @const */ var ns = {};
    /** @const */ ns.anInnerNamespace = {};

This tells the Closure Compiler to track the object as a unique, anonymous type. By contrast, annotations like `@const {Object}` and `@type {Object}` tell the compiler to treat it as any Object in the program (including any subtype of Object).

## Enumerations

Objects used to define a set of values should be typed with `@enum`. Even if you don't use the enum name as a type anywhere, the enum definition restricts the object type and allows strong typing of the referenced values.

    /** @enum {ValueType} */
    var enum = {
      KEY: value
    };

## Maps

Objects used as maps for runtime lookups should restrict their key and value types using generics.

    /** @type {!Object<number, number>} */
    var map = {};
    
    /** @const {!Object<string, ValueType>} */
    var map = {
      'key': value
    };