Where `@suppress` annotations are allowed.
* At the top of a file, in a `@fileoverview` jsdoc
* In the JSDoc of a `function`
* For `@suppress{const}` and `@suppress{duplicate}` only, you can place them on an assignment in statement position
* `@suppress {extraRequire}` is valid just before a `goog.require(...);` statement.