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

### Tagged template literals

Functions intended to be used as a template literal tag are annotated basically the same as a normal function declaration. The first parameter must take an `TemplateLitArray` object or subtype. The remaining parameter(s) correspond to the `${...}` substitutions in the template literal.

For example:

``` js
/**
 * @param {!ITemplateArray} templateObj
 * @param {number} sub1
 * @param {string} sub2
 */
function tag(templateObj, sub1, sub2) {}

// ok
tag`... ${3} ...  ${'hi'} ...`;
// error: Function tag: called with 1 argument(s). Function requires at least 3 argument(s) and no more than 3 argument(s).
tag`...`;
// error: Function tag: called with 2 argument(s). Function requires at least 3 argument(s) and no more than 3 argument(s).
tag`... ${3} ...`;
// error: actual parameter 2 of tag does not match formal parameter
//        found   : string
//        required: number
tag`... ${'not a number'} ...  ${'hi'} ...`;
```

The compiler always infers the implicit template object argument as the first argument passed to to `tag`. Remaining `${}` substitutions are treated as additional arguments.

To define a tag function taking an unlimited number of arguments, use `...`:

``` js
/**
 * @param {!ITemplateArray} templateObj
 * @param {...*} rest
 */
function tagMany(templateObj, ...rest) {}

// all ok
tag``;
tag`...`;
tag`... ${3} ...`;
tag`... ${3} ...  ${'hi'} ...`;
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
