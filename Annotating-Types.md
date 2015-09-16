# Annotating declarations and casts

The compiler recognizes `@type` and function declarations (`@param`, `@return`, etc) annotations in two contexts: declarations and casts.

Variable and functions can be declared with either traditional declarations, or inline declarations, which are more concise.

## Examples

### Function declarations

```javascript
    /** @type {function(number):string} */
    function f(x) {return x + ' apples'}
```
or the more concise inline function declaration:
```javascript
    function /** string */ f(/** number */ x) {return x + ' apples'}
```

### Variable declarations

```javascript
    /** @type {string} */
    var x = 'fruit';
```
or the more concise inline var declaration:
```javascript
    var /** string */ x = 'fruit';
```

### Property declarations
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

### Catch declarations

```javascript
    try { 
      ... 
    } catch (/** @type {string} */ e) {
      ...
    }
```


### Type Casts

Type cast precede a parenthesized expression.

```javascript
    var x = /** @type {string} */ (fruit);
```