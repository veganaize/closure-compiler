# Introduction

In the general case, properties can never be removed in JavaScript because the property references can be hidden by various approaches to property enumeration (for-in, Object.keys, etc).  This is an attempt to describe the various ways the compiler will remove properties.

# Details

## Simple Mode

In Simple mode, a property can be removed if:

It is defined on a object literal and the entire object literal is unreferenced.
```JavaScript
    ({foo: 1});  // can remove this.
```
It is defined on a object literal and assigned to local variable, the value does not escape the local scope, and the variable references allow for the property references to be rewritten into variables
```JavaScript
    var x = {prop1: 1, prop2: 2};
    alert(x.prop2); 
```
becomes:
```JavaScript
    var x$prop1 = 1, x$prop2 = 2;  // x$prop1 is unused and can be removed
    alert(x$prop2);
```

## Advanced Mode

In Advanced mode, the compiler starts making assumptions to allow additional property removal.

It does property flattening as described here (https://developers.google.com/closure/compiler/docs/limitations).  Property flattening is aggressive and restricts the kind of JavaScript code that can be written.  Property flattening only occurs on objects defined in global scope.  Once properties are "flattened" to variable declarations and references, it is possible to remove or inline the values. 

It makes a strong assumption that properties defined on a "prototype" or "this" will not be iterated over and thus is a candidate for removal.  Or more specifically that iteration does not require the property be preserved (you can mixin in properties from one prototype to another).
```JavaScript
    /** @constructor */
    function cls() {
       this.x = 1; // removal candidate due to "this" assumption;
    }
    cls.classProp = 2; // removal candidate due to property flattening
    cls.prototype.y = 2; // removal candidate due to "prototype" assumption.
    cls.prototype.z = function() { // removal candidate due to "prototype" assumption.
       // if "z" is unused and can be removed, "x" and "y" can also be removed.
       alert(this.x + this.y);
    }
```