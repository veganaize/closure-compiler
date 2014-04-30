# Annotating declarations and casts

# Introduction

The compiler recognizes @type annotations in two contexts: declarations and casts.


# Details

## Function declarations

{{{ 
/** @type {function():string} */
function f() {return 'str'}
}}}


## Variable declarations

{{{ 
/** @type {string} */
var x = 'fruit';
}}}

or 

    var /** @type {string} */ x = 'fruit';

## Property declarations

    /** @type {string} */
    x.prop = 'fruit';

or

    var x = {
      /** @type {string} */
      prop : 'fruit'
    };

## Catch declarations

{{{ 
try { 
  ... 
} catch (/** @type {string} */ e) {
  ...
}
}}}


## Type Casts

Type cast precede a parenthesized expression.

    var x = /** @type {string} */ (fruit);