## Overview
`@type {Object}` seems like the right type in for a lot of javascript patterns.  However, `Object` is a very loose type in the Closure Compiler type system and there is often a better type.

## Namespaces

Namespace are best typed as simple @const:

`/** @const */ var ns = {};`
`/** @const */ ns.anInnerNamespace = {};`

This lets the Closure Compiler type system to track the object as a unique anonymous type where as typing it as "Object" or the non-nullable variant "!Object", the compiler must treat it as any Object in the program (including any subtype of Object).

## Enumerations

Objects used to define a set of values should be typed as @enums. Even if you don't use the enum name as a type anywhere, the enum definition restricts the object type and allows strong typing of the object.

`/** @enum {ValueType} */`
`var enum = {`
  `KEY: value,`
`};`

## Maps

Objects used as maps with dynamic property lookup or additions should be specified with type arguements so as to restrict the key and value types.

`/** @type {Object<number, number>} */`
`var map = {};`

`/** @const {Object<string, ValueType>} */`
`var map = {`
  `'key': value,`
`};`






