# BigInt support

## Definitions

* The type of a big integer value is called `bigint` by the compiler
* The function for creating them is called `BigInt()`.
* The feature is usually referred to as BigInt.

## Summary of support

The compiler supports use of BigInt, but there are some restrictions.

1. The compiler won't allow you to use `bigint` literals (e.g. `1n`) unless your `--language_out` is set to at least `ECMASCRIPT_2020`.

   This is because that is the first version of the language specification to include them, and the compiler cannot polyfill BigInt values.
   The intention here is to make sure the compiler warns you about this, so it doesn't surprise you at runtime.

1. You can get the compiler to produce output less than `ECMASCRIPT_2020` that uses BigInt, if you only construct BigInt values by using
   the `BigInt()` constructor on your source code. (e.g. `BigInt("1234567890123467890")`).

   You will get runtime errors if you try to run the compiler's output on a platform where BigInt isn't actually supported.
   If you only use `BigInt()` to create values, then it is possible for your code to check whether `BigInt` is defined and avoid using it
   when it's not. The compiler's refusal to transpile literals ensures that you don't forget to do this.

1. If you specify a `--language_out` lower than `ECMASCRIPT_2016` (when `**` was added to the language),
   and you use `bigint` values with `**` operations, one of these things will happen.

   * The compiler will stop with an error because it noticed you were doing this.
   * The compiler will not notice you were doing this (because you didn't have type checking enabled,
     or there was insufficient or incorrect type information), and will incorrectly transpile the `x ** y` to `Math.pow(x, y)`.

1. Make sure you explicitly annotate variables, parameters, and return types that are supposed to hold `bigint` values.

   If type information is missing, the compiler will assume that arithmetic operations still result in `number` values.
   It's an error to mix `number` and `bigint` values in arithmetic operations.
   See more details below.

## There is no BigInt polyfill

It is not practical to provide a polyfill for `BigInt`, so if you use it, you must be certain that the platform where your code will run will really support it.

The compiler will warn you about this in the following ways.

* If the output language is lower than `ECMASCRIPT_2020`,
  the compiler will report an error and stop compilation the input source code uses BigInt literal syntax (e.g. `1n`).

* If the output language is lower than `ECMASCRIPT_2016`
  (requires transpilation of `x ** y` operators to `Math.pow(x, y)`),
  the compiler will report and stop compilation if the source contains a `**` operator with a BigInt operand.
  (Note that the compiler can only notice this if type checking is enabled and type annotations are sufficient and correct.)

* If you wish to use BigInt with an output language lower than `ECMASCRIPT_2020`,
  then you must avoid using the literal syntax and construct all of your values by calling the BigInt constructor,
  e.g. `BigInt("12345678901234567890")`.

  This strategy has already been used by several projects along with custom BigInt externs files defining BigInt to be equivalent to number,
  before the compiler added BigInt support. Custom definitions of BigInt in project-specific externs files will now conflict with those in
  the default externs, so they should no longer be used.

## Type checking caveats

### It is an error to mix `bigint` and `number` in arithmetic operations.

The compiler will report an error for arithmetic operations where one operand is a `bigint` and the other is not a `bigint`.

It is a runtime error to apply an arithmetic operation to a `bigint` and a non-`bigint` value,
so the compiler will consider it a compile error to do so.
This includes the case where one operand is `bigint` and the other is `bigint|number`. 

```javascript
3n * 5; // error
3n * someUnknownType; // error
3n * BigInt(5); // OK
```

**EXCEPTION:** You can use `+` to concatenate a `bigint` and a `string`.

**NOTE:** Unsigned right shift (`>>>` and `>>>=`) are always illegal for `bigint` according to the language specification.

### Explicitly use the type `bigint|number` to write code that works with both.

The compiler will allow arithmetic operations when both operands are `bigint|number` and infer the result to be `bigint|number`.

If both operands are `bigint|number`, the compiler will not report a type error, because it will assume that the code author has taken responsibility for making sure that at runtime they are either both `bigint` or both `number`.
This is to allow for code that is intentionally designed to work with either `bigint` or `number` values.

```javascript
/**
 * @param {bigint|number} x
 * @param {bigint|number} y
 * @return {bigint|number}
 */
function mul(x, y) {
  return x * y;
}
```

You will need to cast a `bigint|number` value explicitly to one or the other
in order to use it in an operation with a `bigint` or a `number`. e.g.

```javascript
2n * mul(25n, 18n); // ERROR: mixing bigint with bigint|number
2n * /** @type {bigint} */ (mul(25n, 18n)); // OK
```

### Arithmetic operations still return `number` when type information is lacking.

When there isn't sufficient type information, arithmetic operations are assumed to have a result of type `number`.

Before the existence of BigInt the compiler could safely assume that the result of arithmetic operations (e.g. `x * y`) was guaranteed to be of type number, even when the types of the operands were unknown.
If we were pedantic about it we would have to say that the results of such operations are now `bigint|number` unless we have enough type information to say otherwise, but this would likely be incorrect in 99% of cases, and would likely lead to confusing error messages.

```javascript
// If this is supposed to work with `bigint` values, then it needs to have type annotations saying so.
// Otherwise, you're going to have a bad time.
function mul(x, y) {
  return x * y; // type is number, since x and y have unknown types.
}
```