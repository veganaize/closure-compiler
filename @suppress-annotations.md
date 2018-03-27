If you cannot easily fix a JsCompiler warning, you can suppress it using the `@suppress` annotation in the function's JSDoc block or the `@fileoverview` block or any assignment in statement position.

## Example
```js
/**
 * ...
 * Good; suppresses within the entire function.
 * Also, this suppresses multiple warnings.
 * @suppress {visibility|underscore} 
 */
function blah() {
  /** @suppress {visibility} */ // Good; suppresses within this declaration only.
  otherClass.privateField_ = blah;

  /** @suppress {checkTypes} */  // Bad; doesn't work.
  foo(incompatible.type);

  /** @suppress {missingRequire} */  // Good; this is an exception (see below).
  foo(blah.not.required);
}
```

## Other locations
Some specific `@suppress` annotations are allowed elsewhere:

* `@suppress {extraRequire}` is valid just before a `goog.require(...);` statement.
* `@suppress {switch}` is valid on a  switch statement.
* `@suppress {missingRequire}` is valid on any statement.

All other annotations can only be suppressed for a function, file, or declaration.  You cannot suppress most warnings for a single line within a function.


## Suppression tags
The full list of suppressions accepted by the parser can be found in the `jsdoc.suppressions` list [here](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/parsing/ParserConfig.properties#L147).

The specific warnings included in each suppression can be found [here](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroups.java) (note that not all groups are recognized as suppressions)