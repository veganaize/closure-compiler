If you cannot easily fix a JsCompiler warning, you can suppress it using the `@suppress` annotation in the function's JSDoc block or the `@fileoverview` block.


## Other locations
Some specific `@suppress` annotations are allowed elsewhere:

* `@suppress {const}` and `@suppress {duplicate}` are valid on an assignment in statement position.
* `@suppress {extraRequire}` is valid just before a `goog.require(...);` statement.
* `@suppress {missingRequire}` is valid on any statement.

All other annotations can only be suppressed for a function or file.  You cannot suppress most warnings for a single line within a function.

## Example
```js
/**
 * ...
 * @suppress {visibility} // Good; suppresses within the entire funciton
 */
function blah() {
  /** @suppress {visiblity} */ // Bad; doesn't work.
  otherClass.privateField_ = blah;

  /** @suppress {const} */  // Good; this is an exception.
  SOME_CONSTANT = blah;
}
```

## Suppression tags
The full list of suppressions accepted by the parser can be found in the `jsdoc.suppressions` list [here](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/parsing/ParserConfig.properties#L147).

The specific warnings included in each suppression can be found [here](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroups.java) (note that not all groups are recognized as suppressions)