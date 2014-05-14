# Annotating declarations and casts

# Introduction

The compiler recognizes @type annotations in two contexts: declarations and casts.


# Details

## Function declarations

```javascript
/** @type {function():string} */
function f() {return 'str'}
```


## Variable declarations

```javascript
/** @type {string} */
var x = 'fruit';
```

or 

```javascript
    var /** @type {string} */ x = 'fruit';
```

## Property declarations
```javascript
    /** @type {string} */
    x.prop = 'fruit';
```
or

```javascript
    var x = {
      /** @type {string} */
      prop : 'fruit'
    };
```

## Catch declarations

```javascript
try { 
  ... 
} catch (/** @type {string} */ e) {
  ...
}
```


## Type Casts

Type cast precede a parenthesized expression.

```javascript
    var x = /** @type {string} */ (fruit);
```