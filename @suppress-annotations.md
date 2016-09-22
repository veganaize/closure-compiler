Where `@suppress` annotations are allowed.
* At the top of a file, in a `@fileoverview` jsdoc
* In the JSDoc of a `function`
* For `@suppress {const}` and `@suppress {duplicate}` only, you can place them on an assignment in statement position
* `@suppress {extraRequire}` is valid just before a `goog.require(...);` statement.
* `@suppress {missingRequire}` is valid on any statement.

In particular, you cannot suppress most warnings for a single line within a function.

## Suppression tags
The full list of suppressions accepted by the parser can be found in the `jsdoc.suppressions` list [here](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/parsing/ParserConfig.properties#L147).

The specific warnings included in each suppression can be found [here](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroups.java) (note that not all groups are recognized as suppressions)