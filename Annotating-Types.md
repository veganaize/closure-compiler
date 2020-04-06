# Annotating declarations and casts

The compiler recognizes `@type` and function declarations (`@param`, `@return`, etc) annotations in two contexts: declarations and casts.

Variable and functions can be declared with either traditional declarations, or inline declarations, which are more concise.

For more information about specific types that can be used in @type annotations see:
[Types in the Closure Type System](https://github.com/google/closure-compiler/wiki/Types-in-the-Closure-Type-System)

## Examples

### Variable declarations

```js
/** @type {string} */
let x = 'fruit';
```
or the more concise inline var declaration:
```js
let /** string */ x = 'fruit';
```

Variables declared through destructuring can only be typed with an inline var declaration:
```js
const {/** string */ x, /** number */ y} = obj;
```

### Function declarations

```js
/**
 * @param {number} x
 * @return {string}
 */
function f(x) { return x + ' apples'; }
```

or the more concise inline function declaration:
```js
function /** string */ f(/** number */ x) {return x + ' apples'}
```
or as a complete type:

```js
/** @type {function(number):string} */
function f(x) {return x + ' apples'}
```

### Property declarations
```js
/** @type {string} */
x.prop = 'fruit';
```
or

```js
let x = {
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
let x = /** @type {string} */ (fruit);
```

Typedef
--------

Complex types (including unions, and record types) can be aliased for convenience and maintainability using a typedef. These annotations can be long, but can be split over multiple lines for readability.

```js
/** 
 * @typedef {{
 *            foo:string,
 *            bar:number,
 *            foobar: (number|string)
 *          }}
 */
let MyType;

/** @param {MyType} bar */
function foo(bar) {}
```
