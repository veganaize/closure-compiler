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
The full list of suppressions accepted by the parser can be found in the `jsdoc.suppressions` list [here](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/parsing/ParserConfig.properties#L154).

The specific warnings included in each suppression can be found [here](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroups.java) (note that not all groups are recognized as suppressions).

### Error to suppression map

This table maps all suppressible error ids to the corresponding `@suppress` tag. Note that some errors are not suppressible in code and are omitted from this table.

These suppression tags are not canonical. In some cases multiple tags may suppress the same error.

| Error | Suppression tag |
|---|---|
|BAD_REST_PARAMETER_ANNOTATION|misplacedTypeAnnotation|
|JSC_ABSTRACT_METHOD_IN_CONCRETE_CLASS|checkTypes|
|JSC_ABSTRACT_METHOD_NOT_IMPLEMENTED|checkTypes|
|JSC_ABSTRACT_SUPER_METHOD_NOT_USABLE|checkTypes|
|JSC_ARGUMENTS_ASSIGNMENT|es5Strict|
|JSC_ARGUMENTS_CALLEE_FORBIDDEN|es5Strict|
|JSC_ARGUMENTS_CALLER_FORBIDDEN|es5Strict|
|JSC_ARGUMENTS_DECLARATION|es5Strict|
|JSC_ARROW_FUNCTION_AS_CONSTRUCTOR|misplacedTypeAnnotation|
|JSC_BAD_JSDOC_ANNOTATION|nonStandardJsDocs|
|JSC_BAD_PACKAGE_PROPERTY_ACCESS|visibility|
|JSC_BAD_PRIVATE_GLOBAL_ACCESS|visibility|
|JSC_BAD_PRIVATE_PROPERTY_ACCESS|visibility|
|JSC_BAD_PROTECTED_PROPERTY_ACCESS|visibility|
|JSC_BAD_TYPES_FOR_BINARY_OPERATION|checkTypes|
|JSC_BAD_TYPE_FOR_BIT_OPERATION|checkTypes|
|JSC_BAD_TYPE_FOR_UNARY_OPERATION|checkTypes|
|JSC_CLASS_DISALLOWED_JSDOC|lintChecks|
|JSC_CLOSURE_CALL_CANNOT_BE_ALIASED_ERROR|closureDepMethodUsageChecks|
|JSC_COMMONJS_SUSPICIOUS_EXPORTS_ASSIGNMENT|moduleLoad|
|JSC_COMMONJS_UNKNOWN_REQUIRE_ENSURE_ERROR|moduleLoad|
|JSC_COMPUTED_PROP_NAME_IN_ENUM|lintChecks|
|JSC_CONFLICTING_EXTENDED_TYPE|checkTypes|
|JSC_CONFLICTING_IMPLEMENTED_TYPE|checkTypes|
|JSC_CONSTANT_PROPERTY_DELETED|constantProperty|
|JSC_CONSTANT_PROPERTY_REASSIGNED_VALUE|constantProperty|
|JSC_CONSTANT_REASSIGNED_VALUE_ERROR|const|
|JSC_CONSTRUCTOR_NOT_CALLABLE|checkTypes|
|JSC_CONSTRUCTOR_REQUIRED|checkTypes|
|JSC_CTOR_INITIALIZER_NOT_CTOR|checkTypes|
|JSC_DEBUGGER_STATEMENT_PRESENT|checkDebuggerStatement|
|JSC_DECLARE_LEGACY_NAMESPACE_IN_NON_MODULE|lintChecks|
|JSC_DEFAULT_EXPORT|lintChecks|
|JSC_DEFAULT_EXPORT_IN_GOOG_MODULE|lintChecks|
|JSC_DEFAULT_PARAM_MUST_BE_MARKED_OPTIONAL|misplacedTypeAnnotation|
|JSC_DELETE_VARIABLE|es5Strict|
|JSC_DEPRECATED_CLASS|deprecated|
|JSC_DEPRECATED_CLASS_REASON|deprecated|
|JSC_DEPRECATED_PROP|deprecated|
|JSC_DEPRECATED_PROP_REASON|deprecated|
|JSC_DEPRECATED_VAR|deprecated|
|JSC_DEPRECATED_VAR_REASON|deprecated|
|JSC_DETERMINISTIC_TEST|suspiciousCode|
|JSC_DISALLOWED_MEMBER_JSDOC|misplacedTypeAnnotation|
|JSC_DUPLICATE_CASE|suspiciousCode|
|JSC_DUPLICATE_ENUM_VALUE|lintChecks|
|JSC_DUPLICATE_IMPORT|lintChecks|
|JSC_DUPLICATE_MEMBER|es5Strict|
|JSC_DUPLICATE_PARAM|es5Strict|
|JSC_DUP_VAR_DECLARATION|duplicate|
|JSC_DUP_VAR_DECLARATION_TYPE_MISMATCH|duplicate|
|JSC_ENUM_DUP|checkTypes|
|JSC_ENUM_PROP_NOT_CONSTANT|lintChecks|
|JSC_ENUM_VALUE_NOT_STRING_OR_NUMBER|lintChecks|
|JSC_ES5_CLASS_EXTENDING_ES6_CLASS|checkTypes|
|JSC_EVAL_ASSIGNMENT|es5Strict|
|JSC_EVAL_DECLARATION|es5Strict|
|JSC_EXPECTED_THIS_TYPE|checkTypes|
|JSC_EXTENDS_NON_OBJECT|checkTypes|
|JSC_EXTENDS_WITHOUT_TYPEDEF|checkTypes|
|JSC_EXTEND_FINAL_CLASS|visibility|
|JSC_EXTERNS_FILES_SHOULD_BE_ANNOTATED|lintChecks|
|JSC_EXTRA_REQUIRE_WARNING|extraRequire|
|JSC_FRACTIONAL_BITWISE_OPERAND|transitionalSuspiciousCodeWarnings|
|JSC_FUNCTION_ARGUMENTS_PROP_FORBIDDEN|es5Strict|
|JSC_FUNCTION_CALLER_FORBIDDEN|es5Strict|
|JSC_FUNCTION_MASKS_VARIABLE|duplicate|
|JSC_GOOG_MODULE_INVALID_GET_CALL_SCOPE|closureDepMethodUsageChecks|
|JSC_GOOG_MODULE_IN_NON_MODULE|lintChecks|
|JSC_HIDDEN_INTERFACE_PROPERTY|missingOverride|
|JSC_HIDDEN_INTERFACE_PROPERTY_MISMATCH|checkTypes|
|JSC_HIDDEN_PROTOTYPAL_SUPERTYPE_PROPERTY_MISMATCH|checkPrototypalTypes|
|JSC_HIDDEN_SUPERCLASS_PROPERTY|missingOverride|
|JSC_HIDDEN_SUPERCLASS_PROPERTY_MISMATCH|checkTypes|
|JSC_IFACE_INITIALIZER_NOT_IFACE|checkTypes|
|JSC_ILLEGAL_CLASS_KEY|checkTypes|
|JSC_ILLEGAL_IMPLICIT_CAST|checkTypes|
|JSC_ILLEGAL_OBJLIT_KEY|checkTypes|
|JSC_ILLEGAL_PROPERTY_ACCESS|checkTypes|
|JSC_ILLEGAL_PROPERTY_CREATION|checkTypes|
|JSC_ILLEGAL_PROTOTYPE_MEMBER|lintChecks|
|JSC_IMPLEMENTS_NON_INTERFACE|checkTypes|
|JSC_IMPLEMENTS_WITHOUT_CONSTRUCTOR|checkTypes|
|JSC_INCOMPATIBLE_EXTENDED_PROPERTY_TYPE|checkTypes|
|JSC_INCORRECT_PARAM_NAME|lintChecks|
|JSC_INCORRECT_SHORTNAME_CAPITALIZATION|lintChecks|
|JSC_INEXISTENT_ENUM_ELEMENT|checkTypes|
|JSC_INEXISTENT_PARAM|checkTypes|
|JSC_INEXISTENT_PROPERTY|missingProperties|
|JSC_INEXISTENT_PROPERTY_WITH_SUGGESTION|missingProperties|
|JSC_INSTANTIATE_ABSTRACT_CLASS|checkTypes|
|JSC_INSUFFICIENT_OUTPUT_VERSION|missingPolyfill|
|JSC_INTERFACE_CLASS_NONSTATIC_METHOD_NOT_EMPTY|lintChecks|
|JSC_INTERFACE_CONSTRUCTOR_SHOULD_NOT_TAKE_ARGS|lintChecks|
|JSC_INTERFACE_DEFINED_WITH_EXTENDS|lintChecks|
|JSC_INTERFACE_METHOD_NOT_EMPTY|checkTypes|
|JSC_INTERFACE_METHOD_NOT_IMPLEMENTED|checkTypes|
|JSC_INTERFACE_METHOD_OVERRIDE|checkTypes|
|JSC_INVALID_ASYNC_RETURN_TYPE|checkTypes|
|JSC_INVALID_CAST|invalidCasts|
|JSC_INVALID_CLOSURE_CALL_ERROR|closureDepMethodUsageChecks|
|JSC_INVALID_DEFINE_VALUE|missingSourcesWarnings|
|JSC_INVALID_INTERFACE_MEMBER_DECLARATION|checkTypes|
|JSC_INVALID_MODIFIES_ANNOTATION|misplacedTypeAnnotation|
|JSC_INVALID_NO_SIDE_EFFECT_ANNOTATION|misplacedTypeAnnotation|
|JSC_INVALID_OCTAL_LITERAL|es5Strict|
|JSC_INVALID_OPERAND_TYPE|strictPrimitiveOperators|
|JSC_INVALID_PARAM|nonStandardJsDocs|
|JSC_IN_USED_WITH_STRUCT|checkTypes|
|JSC_JSDOC_IMPORT_TYPE_WARNING|nonStandardJsDocs|
|JSC_JSDOC_IN_BLOCK_COMMENT|nonStandardJsDocs|
|JSC_JSDOC_MISSING_BRACES_WARNING|lintChecks|
|JSC_JSDOC_ON_RETURN|misplacedTypeAnnotation|
|JSC_JS_MODULE_LOAD_WARNING|moduleLoad|
|JSC_LATE_PROVIDE_ERROR|lateProvide|
|JSC_LENDS_ON_NON_OBJECT|checkTypes|
|JSC_LET_CLOSURE_IMPORT|lintChecks|
|JSC_MALFORMED_REGEXP|checkRegExp|
|JSC_MAYBE_ACCIDENTAL_DEFAULT_EXPORT_IN_GOOG_MODULE|lintChecks|
|JSC_MISPLACED_ANNOTATION|misplacedTypeAnnotation|
|JSC_MISPLACED_MSG_ANNOTATION|misplacedTypeAnnotation|
|JSC_MISSING_CONST_ON_CONSTANT_CASE|lintChecks|
|JSC_MISSING_EXTENDS_TAG|checkTypes|
|JSC_MISSING_JSDOC|lintChecks|
|JSC_MISSING_JSDOC_IN_DECLARATION_STATEMENT|lintChecks|
|JSC_MISSING_MODULE_OR_PROVIDE|missingSourcesWarnings|
|JSC_MISSING_MODULE_OR_PROVIDE_FOR_FORWARD_DECLARE|missingProvide|
|JSC_MISSING_NAMESPACE_IMPORT|moduleLoad|
|JSC_MISSING_NULLABILITY_MODIFIER_JSDOC|lintChecks|
|JSC_MISSING_PARAMETER_JSDOC|lintChecks|
|JSC_MISSING_REQUIRE|missingRequire|
|JSC_MISSING_REQUIRE_IN_PROVIDES_FILE|missingRequire|
|JSC_MISSING_REQUIRE_TYPE|missingRequire|
|JSC_MISSING_REQUIRE_TYPE_IN_PROVIDES_FILE|missingRequire|
|JSC_MISSING_RETURN_JSDOC|lintChecks|
|JSC_MISSING_RETURN_STATEMENT|missingReturn|
|JSC_MISSING_SEMICOLON|lintChecks|
|JSC_MIXED_PARAM_JSDOC_STYLES|lintChecks|
|JSC_MSG_HAS_NO_DESCRIPTION|messageConventions|
|JSC_MSG_HAS_NO_TEXT|messageConventions|
|JSC_MSG_HAS_NO_VALUE|messageConventions|
|JSC_MSG_KEY_DUPLICATED|messageConventions|
|JSC_MSG_NOT_INITIALIZED_USING_NEW_SYNTAX|messageConventions|
|JSC_MSG_TREE_MALFORMED|messageConventions|
|JSC_MULTIPLE_VAR_DEF|checkTypes|
|JSC_MUST_COME_BEFORE_IN_ES6_MODULE|lintChecks|
|JSC_MUTATED_EXPORT|lintChecks|
|JSC_NAMESPACE_REDEFINED|duplicate|
|JSC_NAME_REFERENCE_IN_EXTERNS|externsValidation|
|JSC_NON_DECLARATION_STATEMENT_IN_INTERFACE|lintChecks|
|JSC_NON_STATIC_INITIALIZER_STRING_VALUE_IN_ENUM|lintChecks|
|JSC_NON_STRINGIFIABLE_OBJECT_KEY|checkTypes|
|JSC_NOT_A_CONSTRUCTOR|checkTypes|
|JSC_NOT_FUNCTION_TYPE|checkTypes|
|JSC_NULL_MISSING_NULLABILITY_MODIFIER_JSDOC|lintChecks|
|JSC_OPTIONAL_ARG_AT_END|checkTypes|
|JSC_OPTIONAL_PARAM_NOT_MARKED_OPTIONAL|lintChecks|
|JSC_PARTIAL_NAMESPACE|partialAlias|
|JSC_POLYMER_DESCRIPTOR_NOT_VALID|polymer|
|JSC_POSSIBLE_INEXISTENT_PROPERTY|missingProperties|
|JSC_PREFER_BACKTICKS_TO_AT_SIGN_CODE|lintChecks|
|JSC_PRIMITIVE_OBJECT|lintChecks|
|JSC_PRIMITIVE_OBJECT_DECLARATION|lintChecks|
|JSC_PRIVATE_OVERRIDE|visibility|
|JSC_PROTOTYPAL_HIDDEN_SUPERCLASS_PROPERTY|checkPrototypalTypes|
|JSC_PROVIDES_NOT_SORTED|lintChecks|
|JSC_REASSIGNED_CONSTANT_CASE_NAME|lintChecks|
|JSC_REDECLARED_VARIABLE|checkVars|
|JSC_REDUNDANT_NULLABILITY_MODIFIER_JSDOC|lintChecks|
|JSC_REFERENCE_BEFORE_DECLARE|checkVars|
|JSC_REGEXP_REFERENCE|checkRegExp|
|JSC_REQUIRES_NOT_SORTED|lintChecks|
|JSC_RESOLVED_TAG_EMPTY|missingSourcesWarnings|
|JSC_SAME_INTERFACE_MULTIPLE_IMPLEMENTS|checkTypes|
|JSC_SHORTHAND_ASSIGNMENT_IN_ENUM|lintChecks|
|JSC_STATIC_MEMBER_FUNCTION_IN_INTERFACE_CLASS|lintChecks|
|JSC_STRICT_INEXISTENT_PROPERTY|strictMissingProperties|
|JSC_STRICT_INEXISTENT_PROPERTY_WITH_SUGGESTION|strictMissingProperties|
|JSC_STRICT_INEXISTENT_UNION_PROPERTY|strictMissingProperties|
|JSC_STRICT_MODULE_DEPENDENCY|strictModuleDepCheck|
|JSC_SUSPICIOUS_IN|suspiciousCode|
|JSC_SUSPICIOUS_INSTANCEOF_LEFT|suspiciousCode|
|JSC_SUSPICIOUS_LEFT_OPERAND_OF_LOGICAL_OPERATOR|suspiciousCode|
|JSC_SUSPICIOUS_NAN|suspiciousCode|
|JSC_SUSPICIOUS_NEGATED_LEFT_OPERAND_OF_IN_OPERATOR|suspiciousCode|
|JSC_SUSPICIOUS_SEMICOLON|suspiciousCode|
|JSC_TEMPLATE_TRANSFORMATION_ON_CLASS|checkTypes|
|JSC_TEMPLATE_TYPE_DUPLICATED|checkTypes|
|JSC_TEMPLATE_TYPE_EXPECTED|checkTypes|
|JSC_TEMPLATE_TYPE_ILLEGAL_BOUND|checkTypes|
|JSC_THIS_TYPE_NON_OBJECT|checkTypes|
|JSC_TYPE_MISMATCH|checkTypes|
|JSC_TYPE_ON_GETTER_SETTER|lintChecks|
|JSC_TYPE_PARSE_ERROR|checkTypes|
|JSC_TYPE_REDEFINITION|checkTypes|
|JSC_UNDEFINED_EXTERN_VAR_ERROR|externsValidation|
|JSC_UNDEFINED_VARIABLE|undefinedVars|
|JSC_UNKNOWN_DEFINE_WARNING|unknownDefines|
|JSC_UNKNOWN_EXPR_TYPE|reportUnknownTypes|
|JSC_UNKNOWN_LENDS|checkTypes|
|JSC_UNKNOWN_OVERRIDE|checkTypes|
|JSC_UNKNOWN_PROTOTYPAL_OVERRIDE|checkPrototypalTypes|
|JSC_UNKNOWN_TYPEOF_VALUE|checkTypes|
|JSC_UNREACHABLE_CODE|uselessCode|
|JSC_UNRECOGNIZED_TYPE_ERROR|checkTypes|
|JSC_UNTRANSPILABLE|untranspilableFeatures|
|JSC_UNUSED|underscore|
|JSC_UNUSED_LABEL|lintChecks|
|JSC_UNUSED_LOCAL_ASSIGNMENT|unusedLocalVariables|
|JSC_UNUSED_PRIVATE_PROPERTY|unusedPrivateMembers|
|JSC_USED_GLOBAL_THIS|globalThis|
|JSC_USELESS_BLOCK|lintChecks|
|JSC_USELESS_CODE|uselessCode|
|JSC_USELESS_EMPTY_STATEMENT|lintChecks|
|JSC_USELESS_USE_STRICT_DIRECTIVE|lintChecks|
|JSC_USE_OF_GOOG_PROVIDE|useOfGoogProvide|
|JSC_USE_OF_WITH|es5Strict|
|JSC_VAR|lintChecks|
|JSC_VAR_ARGS_MUST_BE_LAST|checkTypes|
|JSC_VAR_MULTIPLY_DECLARED_ERROR|checkVars|
|JSC_VISIBILITY_MISMATCH|visibility|
|JSC_WRONG_ARGUMENT_COUNT|checkTypes|
|JSC_WRONG_NUMBER_OF_PARAMS|lintChecks|
|JSC_XMODULE_REQUIRE_ERROR|strictModuleDepCheck|
|MODULE_NAMESPACE_MISMATCHES_TYPESCRIPT_NAMESPACE|lintChecks|