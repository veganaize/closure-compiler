# Annotating declarations and casts

The compiler recognizes `@type` and function declarations (`@param`, `@return`, etc) annotations in two contexts: declarations and casts.

Variable and functions can be declared with either traditional declarations, or inline declarations, which are more concise.

## Examples

### Function declarations

```js
/** @type {function(number):string} */
function f(x) {return x + ' apples'}
```
or the more concise inline function declaration:
```js
function /** string */ f(/** number */ x) {return x + ' apples'}
```

### Variable declarations

```js
/** @type {string} */
var x = 'fruit';
```
or the more concise inline var declaration:
```js
var /** string */ x = 'fruit';
```

### Property declarations
```js
/** @type {string} */
x.prop = 'fruit';
```
or

```js
var x = {
  /** @type {string} */
  prop : 'fruit'
};
```

### Catch declarations

```js
try { 
  ... 
} catch (/** @type {string} */ e) {
  ...
}
```

### Type Casts

Type cast precede a parenthesized expression.

```js
var x = /** @type {string} */ (fruit);
```