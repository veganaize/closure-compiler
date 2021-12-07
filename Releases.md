## Introduction

Here's a list of recent releases of Closure Compiler.
See the [README](https://github.com/google/closure-compiler#getting-started) for how to obtain the latest release.
We also update the source distribution and the compile service (at http://closure-compiler.appspot.com) at each release.

The Closure Compiler team's goal is to release during the first week of every month, although we may miss that deadline due to unexpected internal test failures or holidays.

For complete list of changes refer to the [change log](https://github.com/google/closure-compiler/commits/master).

## Details
### December 1, 2021 (v20211201)
*   Remove `GlobalVarReferenceMap` which was part of now obsolete hotswap
    checking support.
*   Deleted methods from JSTypeRegistry that supported hotswap typechecking, as
    hotswap passes are no longer supported in the compiler.
*   Moved injection of transpilation and polyfill libraries just before
    optimizations and after typechecking.
*   In conformance, treat `elem[prop] = foo;` as `elem.setAttribute(prop,
    foo);`, if `elem` is an `Element`.
*   Change ConvertToDottedProperties to include static keyword in computed
    method definition and computed field definition
*   Improve pretty printing for else-if statements
    ```javascript
    // Input
    if (0) {
      0;
    } else if (1) {
      1;
    }

    // Output before:
    if (0) {
      0;
    } else {
      if (1) {
        1;
      }
    }

    // Output after:
    if (0) {
      0;
    } else if (1) {
      1;
    }
    ```
*   Source maps now refer to runtime library and polyfill code by their path
    instead of a name with a `[synthetic:` prefix
*   Modify the `typeof` guard created by CrossChunkCodeMotion to ensure it
    doesn't match dom elements added the global namespace by browsers.
*   Removed @deprecated method `SourceFile.Builder.buildFromFile`. Use
    `builder.withPath(fileName).build()` instead.
*   Add support for --broswer_featureset_year=2018
*   Show JSC_UNKNOWN_DEFINE_WARNING when 'use_typed_ast = True' in js_binary.

### November 7, 2021 (v20211107)

*   ECMASCRIPT_NEXT is now the default language out
    This makes it so that folks need to explicitly ask for transpilation by
    setting the `--language_out` flag. For folks who want the old behavior, you
    can continue to get it by setting `--language_OUT=ECMASCRIPT5` or
    `--language_out=STABLE`.
    Note that in order to maintain the default behavior of not including the
    `"use strict"` directive the `--emit_use_strict` flag now defaults to
    `false`.
*   Fix [incorrect ordering of side-effects](https://github.com/google/closure-compiler/issues/3874)
    when an function inlining target was part of a call target expression.
*   Making ConvertToDottedProperties convert computed properties, member
    functions, and optional chaining with brackets.
*   Change to allow Collapse Properties to collapse properties of functions used
    in call target indirection (i.e. `(0,qualified.name.fn)()`).
    This addresses a code size regression seen with TypeScript 4.4 generated
    code.
*   Fixed regression that caused a crash when setting `chunk_output_type` to
    `ES_MODULES`
*   Fix incorrect documentation of "print_file_after_each_pass" and
    "print_module_after_each_file" flag.
*   Stop rewriting async `super.function()` calls with `Object.getPrototypeOf`.
    This is in effect a rollback of the functional change in
    https://github.com/google/closure-compiler/pull/3103
    Preserving the `super.function()` syntax is more spec-compliant in some
    cases, and MS Edge 17 is incredibly rare these days.
*   Transpile Rest Arguments using a helper function to allow them to be removed
    as dead code.
*   Do not mark indirect call targets for tagged template literals as useless
    code. Indirecting calls in the form `(0, prefix.myFn)`abc` prevents passing
    prefix as the this context object to myFn, which is useful. This fixes a
    dead code removal with TypeScript 4.4, which uses this pattern for functions
    imported from modules.
*   When the special first parameter is unused, convert all tagged template
    literal references into ordinary function calls.
    This allows for debugging or logging functions intended to be called with
    tagged template literals that do nothing when compiled for production to be
    recognized and removed.
*   Fix to the problem where CodePrinter omits required parens around arrow
    function in some contexts.
*   The zero, comma pattern introduced in TS4.4 is recognized in goog.testSuite
    calls. (e.g. `(0, goog.testSuite)({});`)
*   Errors for overriding a `@final` method may no longer be suppressed via
    `@suppress {const}` or `@suppress {constantProperty}`. The canonical way is
    `@suppress {visibility}`.


### October 6, 2021 (v20211006)

*   Crash in Closure Compiler if languageOut is incompatible with ZoneJS. This
    only happens when the compiler detects ZoneJS as one of the inputs. This is
    because ZoneJS is incompatible with async functions (see
    https://github.com/angular/angular/issues/31730)
*   Added `--assume_static_inheritance_is_not_used` flag. It is on by default
    to be consistent with the previous behavior of the compiler. Setting
    it to false will make the compiler expect to see references to static
    methods via `this` in static methods or via subclass names and avoid
    changes that could break that.
*   Don't parse types in JSDoc @throws annotations anymore. Types become part of
    the textual description of the throws annotation.
*   Externs for `Proxy` are now included by default. 
*   Fix output directory of `--print_source_after_each_pass` flag to be
    consistent with released notes.
*   Changed diagnostic type name for overriding final methods from
    `JSC_CONSTANT_PROPERTY_REASSIGNED_VALUE` to `JSC_FINAL_PROPERTY_OVERRIDDEN`.
    The canonical way of suppressing this error is now `@suppress {visibility}`.
    `@suppress {const}` and `@suppress {constProperty}` will still work until we
    finish cleaning up existing usages.
*   Deleted @deprecated `setAliasAllStrings`. Please use
    `setAliasStringsMode(AliasStringsMode.ALL)` to alias all repeated strings,
    which is the same behavior as `setAliasAllStrings(true)`.
*   Added String.raw polyfill.
*   Update public method names in JSDocInfo to be consistent with other record*
    named methods
*   Add support for parsing and printing JSDoc text descriptions after @suppress
    annotation
*   Do not mark indirect calls to functions as useless code.
    Indirecting calls in the form `(0, prefix.myFn)()` prevents passing prefix as
    the this context object to myFn, which is useful. This fixes a dead code
    removal with TypeScript 4.4, which uses this pattern for functions imported
    from modules.


### September 8, 2021 (v20210907)
*   Moved `JSC_UNUSED_PRIVATE_PROPERTY` and `JSC_MISSING_CONST_PROPERTY` checks
    out of the analyzer and into the linter. The DiagnosticGroups
    `UNUSED_PRIVATE_PROPERTY` and `MISSING_CONST_PROPERTY` are deprecated and
    no-ops.

### September 7, 2021 (v20210906)

*   The default input language is updated from ECMASCRIPT_2020 to **ECMASCRIPT_2021.**
*   Fixed bug where using `--rewrite_function_expressions` could break the
    scoping of `this` inside arrow function bodies when not transpiling to ES5.
*   Disallow setting --language_out in conjunction with --browser_featureset_year
*   Correct transpilation of `for await (const [pattern] of something) {}`.
    Previously the compiler crashed for a destructuring variable declaration.
*   Removed lint check for nullable return values that never return null.
*   All transpilation passes now run in stage 2 (a.k.a. the optimizations phase)
    of a multistage compiler build.
*   Add externs for maps api v3.46
*   Deleted @deprecated CompilerOptions.setBrokenClosureRequiresLevel. The only
    thing this did was disable the MISSING_PROVIDE diagnostic group and prevent
    ProcessClosureProvidesAndRequires from deleting goog.requires. The first can
    be done via options.setWarningLevel and the second can be done via
    options.setPreserveClosureProvidesAndRequires.
*   Removed peephole optimization folding `RegExp` calls to regex literals. The
    implementation was complex and had some subtle existing bugs, and the size
    savings measured on sample projects didn't justify the complexity.
*   When pretty printing, print "()" for constructors with empty parameter lists.
*   Fix printing of trailing commas for refactorings.



### August 8, 2021 (v20210808)
* Add a FeatureSet corresponding to the ES2021 language version. Includes features in the ES2021 spec, e.g. logical assignment operators and numeric separators.
* Fixed bug where builds with language_out ES2015+ did not see warning about rest parameter JSDoc missing the ["..." variadic annotation](https://github.com/google/closure-compiler/wiki/Types-in-the-Closure-Type-System).
* Update Maps JS usage allowed externs to allow using the current weekly version (by explicit version number).
* Ban files with nested goog.provides where a nested goog.provide is used as a `@typedef`. If you see a new `JSC_TYPEDEF_CHILD_OF_PROVIDE` error, the recommended fix is to delete the nested goog.provide (and optionally move to goog.module)
* Fix to @const enforcement on @externs properties. A single assignment to the property in executable code now triggers an error.
* Add an error for new proeprties in `metadata:` for Wiz views.
* Fold String#replace when all the values are string literals. Add a few more unit tests to the String#replace folding
* Remove the deprecated `--module` and `--module_wrapper` aliases for the `--chunk` and `--chunk_wrapper` flags.
* Make `JSC_POSSIBLE_INEXISTENT_PROPERTY` error message explain what it actually means.
* Symbol representing namespaces (declared by `oog.provide` or `goog.module`) are prefixed with `ns$` in `SymbolTable`.
* Add polyfills for `Promise.any` and `AggregateError`.
* Switch @fileoverview parsing from "last one wins" to "first one wins". @suppress annotations are merged across all @fileoverviews in a file, including implicit ones (e.g. @externs).
* Running in checksOnly mode no longer runs the pass that replaces `goog.provide('a.b');` with `var a = {}; a.b = {};`. To temporarily revert this change, set `options.setBadRewriteProvidesInChecksOnlyThatWeWantToGetRidOf(true)`.
* Add polyfill and externs for `String.prototype.replaceAll`
* Add externs for maps api v3.45
* Added `GamepadEvent` to `w3c_gamepad.js`
* Added constant folding support for `Math.imul`
* Deleted deprecated CompilerOption `setCheckGlobalNamesLevel` and error group `undefinedNames`, as in `--jscomp_{error,warning,off}=undefinedNames`. These options were no-ops and are covered by other diagnostics. `@suppress {undefinedNames}` still parses but is a no-op.

### June 1, 2021 (v20210601)
*   Dropped `RefasterJS` from the closure-compiler GitHub repo, because it is
    unsupported. See https://github.com/google/closure-compiler/issues/3702
*   Avoid unnecessary polyfills when using browser `FeatureSet` year
    to select the output level.
*   Removed the remains of the "private by convention" wiring in the
    CodeConvention classes and associated lint checks.
*   Compiler option 'setBadRewriteModulesBeforeTypecheckingThatWeWantToGetRidOf'
    is now false by default. This may lead to new type errors. This change only
    affects users of the Java API, not the command line or NPM versions.
*   Deleted the `BanExpose` conformance rule. `@expose` is now a parse error so
    this conformance rule is unnecessary.
*   Removed support for deprecated JSDoc tag `@expose`. Instead either use
    [`@export`](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler#export-export-sometype)
    to fix
    [property renaming issues](https://developers.google.com/closure/compiler/docs/limitations#implications-of-global-variable,-function,-and-property-renaming:),
    or use
    [`@nocollapse`](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler#nocollapse)
    to prevent problems with
    [property flattening](https://developers.google.com/closure/compiler/docs/limitations#implications-of-object-property-flattening).

### May 5, 2021 (v20210505)
*   Deprecated the `undefinedNames` diagnostic group and removed the associated
    error. Undefined namespaces are still detected via enabling `undefinedVars`
    and `missingProperties` diagnostics.
*   Changes to the public command-line runner:
    -   Remove long deprecated flags aliases
        -   `--module` should be `--chunk`
        -   `--module_wrapper` should be `--chunk_wrapper`
        -   `--module_output_path_prefix` should be `--chunk_output_path_prefix`
        -   `--output_module_dependencies` should be
            `--output_chunk_dependencies`
        -   `--common_js_module_path_prefix` should be `--js_module_root`
        -   `--common_js_entry_module` should be `--entry_point`
        -   `--closure_entry_point` should be `--entry_point`
        -   `--module` should be `--chunk`
        -   `--manage_closure_dependencies` should be
            `--dependency_mode=PRUNE_LEGACY`
        -   `--only_closure_dependencies` should be `--dependency_mode=PRUNE`
        -   `--polymer_pass` should be `--polymer_version=1`
    -   Remove long deprecated option `--transform_amd_modules`
    -   Allow dynamic import expressions by default
    -   Document the --json_streams flag
    -   Enable optimal flags automatically when `--chunk_output_type=ES_MODULES`
        is specified
*   `SourceMapConsumerV3`: Add a field to `OriginalMapping` to indicate
    whether the mapping is exact, estimated, or of unknown precision.
*   Canonicalized destructuring import shorthand properties with the JSCompiler
    Linter.
*   Sped up serialization of AST and types in multistage builds. Not expected to
    be a behavioral change.
*   Added BanSetAttribute conformance check
*   For users of the Java API: deleted `SourceFile.fromGenerated`. Replace with
    either a reference to a file on disk or a preloaded string
    `SourceFile.fromCode`

### April 6, 2021 (v20210406)

*   Remove the ability to select specific strings to alias from the compiler
    API.
*   Enable trailing commas in arrow function parameter lists as allowed in the
    language spec:
    https://tc39.es/ecma262/#prod-CoverParenthesizedExpressionAndArrowParameterList
*   Add `--dynamic_import_alias` option which instructs the
    compiler to replace dynamic import expressions with a function call using
    the specified name. This allows dynamic import expressions to be externally
    polyfilled when the output language level does not natively support them.
*   Properly report a parse error for functions that have trailing commas after
    rest parameters.
*   In the AliasString pass remove support for skipping certain strings based on
    whether they match a supplied regex.
*   For AliasString pass, rather than using the source location of the file that
    aliased strings are added to, use instead the first original use location.
    This will prevent the first file of affected chucks from being artificially
    bloated when using source maps to determine "weight" of input source on the
    output.
*   The `BanGlobalVars` conformance rule now accepts specific variables names to
    allowlist via the `value` field.
*   Stricten validation of goog. module ids to match closure/base.js.
    Only ASCII, \_, $, and 0-9 are now allowed.
*   Add linter warning for using `extends` keyword in @interface/@records
    definition.
*   Refactoring of compiler handling of goog.define: tooling using checksOnly
    will now see the original source's `goog.define('name', 0);` instead being
    rewritten to the default value `0`
*   Implement decomposition of `super.method()` calls in `ExpressionDecomposer`.
    This fixed a compiler crash that could occur when inlining a function call
    into the arguments of a `super.method()` call.
*   Fixed enableMultistageCompilation() option in `CompilerTestCase` API to work
    again, and also run multistage compilation if `enableNormalize()` is on.
*   Fixes to optimizations of 'class side inhertiance', including small fixes to
    make optimizations both safer and more consistent.

### March 2, 2021 (v20210302)

*   Bugfix: Allow `--browser_featureset_year=2021` to be passed via the command
    line.
*   Add externs for maps api v3.44
*   Remove support for "private" by convention properties in `CheckProvides`.
    Since "by convention" private is not enforced elsewhere it doesn't make
    sense to enforce it here.
*   Upgrade the default `language_in`/`language_out` settings
    to the stable defaults for users from ant. Also remove the ability to
    override the `language_in` value through ant.
*   Remove the `CheckProvides` compiler checks. This pass would look for
    `@constructor`s that were not not paired with `goog.provide`. The pass had many
    holes and has limited value in a world where `goog.provide` files are being
    actively removed.
*   `GETPROP` and `OPTCHAIN_GETPROP` Nodes now have source info corresponding to the
    source text of their property name, rather than their entire expression
    tree.

### February 2, 2021 (v20210202)

*   Added experimental support for allowing dynamic import expressions
    `import('./path')` to pass through the compiler unchanged. Enabling this
    requires supplying the command line option `--allow_dynamic_import` and also
    setting `--output_language=ES_2020`. See also [#2270](https://github.com/google/closure-compiler/issues/2770#issuecomment-668271474).
*   Fixed bug where `--generate_exports=false` failed to disable the option.
*   Delete flag `--dart_pass`. Assume its default value, `false`.
*   Remove the `strictMissingRequire` and `stricterMissingRequire` diagnostic
    groups completely, in favor of the `missingRequire` flag.
*   Introduce a `BanStaticThis` conformance check for forbidden references to
    `this` in a static context.
*   Deprecate the APIs used to configure the StripCode pass and make the
    CompilerOptions#strip* members package private.
*   Remove support for `CompilerOptions#setStripTypePrefixes()`
*   Switched to new implementation of property disambiguation. This is faster
    for large projects but slightly more conservative.
*   Enable "optimize constructors" when "optimize calls" is enabled.
*   Create an optimization to remove trivial ES6 class constructors where the
    implicit constructor is equivalent.
*   Fixes a [bug](https://github.com/google/closure-compiler/issues/3756) where arrow functions that referenced this could be improperly
    inlined in ES2015 output mode. Thanks @DavidANeil!
*   Compiler no longer disambiguates properties that are ever used on a `@record`.
     This is expected to improve code size in some cases and regress it in others
*   Compiler now assumes that ES5-style prototype method assignments
    `Foo.prototype.m = function() {` never trigger setters. This improves code
    size for projects that use getters/setters.

### January 6, 2021 (v20210106)

*   Added experimental support for allowing dynamic import expressions 
    import('./path') to pass through the compiler unchanged. Enabling
    this requires supplying the command line option --allow_dynamic_import
    and also setting --output_language=ES_2020.
*   Compiler now assumes that ES5-style prototype method assignments 
    Foo.prototype.m = function() { never trigger setters. This improves code
    size for projects that use getters/setters.
*   Remove CodingConvention#getGlobalObject and special handling for treating 
    some deleted goog.* functions as "property checks". 
*   Use the 'missingRequire' diagnostic group (previously a no-op) to turn on
    the new stricter missing require check. From this release, using 
    '--jscomp_error=missingRequire' is the recommended way to turn on missing 
    requires checks.
*   Allow closure bundler to optionally embed source map.
*   Define --browser_featureset_year=2021 based on Chrome 87, Firefox 84, and 
    Safari 14. Almost everything through ES2020, including bigint, optional 
    chaining, and more.
*   GlobalNamespace was treating a lot of operators as potentially creating 
    aliases that should prevent property collapsing. This was a missed 
    optimization opportunity.
*   Always apply the input source maps in the base transpiler. This is useful
    for transpiling generated files from typescript compiler. 
*   Add a new option to embed the source map in base64 encoding (instead of 
    textual escaping).
*   The --generate_exports and --export_local_property_definitions flags now
    default to true. This only affects code that uses the@export annotation 
    but does not already set these flags. 
*   Preserve parentheses on spread operator's AssignmentExpression. Previously,
    these were incorrectly dropped in the compiler's output, creating syntax errors.
*   goog.tweak is deprecated. Turn unused flag --override_tweak_value and 
    switch closure_tweak_processing from CHECK to OFF by default.

### December 7, 2020 (v20201207)

*   Type-based property ambiguation now ambiguates properties accessed on JS
    scalars. This has a negligible effect on code size. It's unliekly to be a
    breaking change since type-based property disambiguation already
    disambiguates scalar properties
*   Fixed https://github.com/google/closure-compiler/issues/3733, which caused
    incorrect collapsing of properties, generating broken code.
*   Add externs for HTMLMediaElement.captureStream.
*   Add externs for CanvasCaptureMediaStreamTrack.
*   Add externs for ImageBitmap.close.
*   Fix externs for RTCTrackEvent to make fields non-optional.
*   Fix externs for RTCPeerConnection.setLocalDescription to make all args
    optional.
*   JsFileFullParser now reports symbols and dependencies in goog.loadModule
    calls.
*   Made function inlining slightly more aggressive, with the result that more
    call sites are inlined. Generally improves code size but we have seen a few
    small increases in code size.
*   Add externs for maps api v3.43
*   Don't conflate async and generator functions with regular
    function in FunctionRewriter.
*   The ReplaceStrings pass no longer supports being passed `.prototype` methods
    in its configuration. Non-prototype namespaced methods are still supported.
*   `OffscreenCanvas` can now be passed to `createImageBitmap`
*   Remove the old "missingRequire", "strictMissingRequire", and
    "legacyGoogScopeRequire" diagnostic groups (and the associated
    CheckMissingAndExtraRequires pass).

    If you would like to continue to get checking of missing requires in your
    binary builds, the recommendation is to use the CheckMissingRequires pass
    currently controlled by the "stricterMissingRequire" diagnostic group.
*   `Window.prototype.navigator` is no longer nullable.
*   Avoid reporting a compilation error for `@async` annotations,
    which are now part of the Open Source JSDoc standard.
    The compiler will ignore these annotations.
*   Omit fill files from the manifest.

### November 2, 2020 (v20201102)
*   Add `$jscomp.global` to the list of known aliases of the global object
*   Delete always-false `isDisposes` method from `JSDocInfo` API
*   Removed a peephole optimization that folds calls to `Array.prototype.concat`
    based on inferred type information. This optimization still applies to array
    literals.
*   `ClosureCodeRemoval`: Limit abstract removal to empty function expressions.

    In some cases the compiler would incorrectly remove assignments annotated as
    `@abstract`. We've restricted it to removing assignments of the special
    `goog.abstractFunction` value and empty function expressions (`function()
    {}`) that are not also annotated with `@constructor`.
*   Inline constant parameters and remove unused ones for ES6 class constructor
    calls as is done for normal functions. This feature was overlooked when ES6
    class support was added.

### October 6, 2020 (v20201006)

*   `ECMASCRIPT_2020` is now the default input language.
*   Add lint warning for using `var` (prefer `let` or `const`).
*   Remove deprecated noop diagnostic group: missingGetCssName
*   Add `updateTiming` option to `AnimationEffect`.
    Animation Effects have a function called updateTiming,
    https://drafts.csswg.org/web-animations-1/#dom-animationeffect-updatetiming.
*   Bazel J2CL rules are deleted.
*   --version flag no longer prints the build time
*   Add `$jscomp.FORCE_POLYFILL_PROMISE_WHEN_NO_UNHANDLED_REJECTION` compiler
    @define. It allows you to force promise polyfill on browsers that natively
    support promise but not the unhandledrejection event.
*   Allow left and right shift operators to apply to bigint values.
*   Revamped type-based property ambiguation to use a heavily simplified
    representation of Closure types (called "colors"). This is not expected to
    affect the optimizer's output.

### September 28, 2020 (v20200927)

*   Add back the ANT plugin code that was dropped when migrating to Bazel.
*   Remove `ENABLE_UNHANDLED_REJECTION_POLYFILL` compiler @define. The feature
    is no longer optional.

### September 21, 2020 (v20200920)

*   **[KNOWN ISSUE]** Does not contain ANT plugin code
*   Support for "name anonymous functions" and its associated options and flags
    have been removed.
*   Type checking support for obsolete Closure Library methods
    goog.isDef/isDefAndNotNull/isNull/isString/isBoolean/isNumber/isFunction/isArray
    has been removed. These methods have been removed from the library.
*   Properties marked as @const on a structural interface are now treated as
    readonly when accessed through the structural type. Note that properties on
    implementing types may be mutable.
*   Unused properties set on non-constructor functions were not previously
    removed, but now they may be.
    <br><br>
    Also, an unused property assignment `X.unusedProp` that was previously
    removed may now be left behind if the value assigned to `X` is not a class
    or function literal. (e.g. `const X = createClassX();`)
    <br><br>
    RemoveUnusedCode attempts to distinguish properties set on constructors from
    those set on other kinds of objects. It may be able to remove properties set
    on constructors if they are never used.
    <br><br>
    Prior to this change, when it considered `X.prop = value`, it would look at
    the JSType information of `X` to determine whether `X` was a constructor.
    However, that is problematic for the future direction we want to take to use
    less type information in optimization passes.
    <br><br>
    With this change, RemoveUnusedCode instead looks at what value was assigned
    to `X`. If the value is a function or class literal, it will treat `X.prop`
    as if it were set on a constructor.
    <br><br>
    This change is expected to make very little difference to the optimized
    output, but there are some subtle differences.
*   Add `unhandledrejection` support in the promise polyfill.
    <br><br>
    The polyfill passes most of the WPT test suite (http://shortn/_OPfqQ0tVC7)
    except for the following cases:
      - `event.promise` is an instance of polyfill promise instead of native
        promise.
      - You cannot attach a rejection handler after waiting for nested promises
        because polyfill promise and unhandledrejection are both delayed with
        `setTimeout`. Delay attaching a rejection handler after microtasks works
        as expected.
*   Back off unsafe property (dis)ambiguation on properties of functions used as
    namespaces, except for constructors and interfaces. This may cause a slight
    code size regression.
*   When pretty printing JSDoc annotations, they now occupy their own line.
*   Add definition for `_.bind()` to underscore-1.4.4.js externs file.
*   The `unhandledrejection` event is now enabled by default.
*   Obsolete `WebWorker` externs have been removed.
    The name was changed to just `Worker` quite some time ago.
*   Add externs for `Navigator.prototype.wakeLock` and associated interfaces,
    `w3c_screen_wake_lock.js`.
*   Remove TypeScript -> Closure JS transpilation, as it is unmaintained and
    does not support newer TS features. https://github.com/angular/tsickle is
    the better-supported tool for TS -> Closure.
*   Passing the experimental `--language_in=ECMASCRIPT6_TYPED` is now forbidden.
    The TS parser has not been kept up-to-date.
*   Stop building Maven artifacts closure-compiler-linter and
    closure-compiler-gwt, and stop uploading them to Sonatype.
*   Fix printing COMMA expression as RHS of DESTRUCTURING_LHS
    <br><br>
    When a variable declaration uses destructuring and the RHS is a comma
    expression, the printer would omit the necessary parentheses. This would
    cause an unexpected `var {foo} = first, second` declaring a new `second`
    binding. Or, it would cause an outright syntax error, eg `var {foo} = first,
    second()`.
*   Add externs for W3C Keyboard Lock API
*   Fix incorrect DCE of calls to fns with destructuring params. See
    https://github.com/google/closure-compiler/issues/3499 for more details.

### August 30, 2020 (v20200830)

* Add externs for Jasmine toBeRejectedWithError()
* Fix property ambiguation bug where properties on structural types were sometimes unsafely renamed. This may slightly increase post-gzip size but is necessary for correctness.
* Add externs for maps api v3.42
* Add externs for `NodeList.prototype.entries()`, `.keys()`, and `.values()` methods.
* Backoff in InlineProperties on structural type mismatches, bringing it more in line with the (dis)ambiguate properties optimizations.
* Property ambiguation no longer accounts for constructors that were dead-code eliminated after typechecking. This shouldn't be a breaking change unless it triggers problems with existing violations of compiler assumptions in typings.
* Support for globalThis and optional chaining in GatherRawExports
* CheckMissingGetCssName has been removed and the associated settings are now noops.
* Allow concatenating compile-time literals via ECMAScript templates (which only substitute compile-time constants) in calls to goog.string.Const.from
* Correctly report the ModuleType of an ES module in JsFileFullParser.
* Omit weak files from single-module manifests.
* BigInt is now supported for `language_in=ECMASCRIPT_2020`. However, it won't be transpiled or polyfilled. If language_out is lower than ES_2020:
  * The compiler will report errors for BigInt literals.
  * The compiler will not report errors for using the BigInt() function to create bigint values, but you will get runtime errors if your code executes in an environment where BigInt is not available.
* CompilerOptions#setExtraSmartNameRemoval() is now a noop. "extra smart name removal" is now always enabled with "smart name removal".
* Prevent --strict from promoting IMPLICIT_WEAK_ENTRY_POINT_ERROR to an error, as this diagnostic is meant to be purely advisory.
* Remove noop flag --remove_unused_prototype_props_in_externs
* Remove noop API CompilerOptions#setRemoveUnusedPrototypePropertiesInExterns


### July 19, 2020 (v20200719)

*   Optimized code no longer contains JSDoc/types on the AST. For use-cases that
    still require output to have types, please use the `checksOnly` mode.
*   Rename AbstractCommandLineRunner#setWarningsWhitelistFile to
    setWarningsAllowlistFile
*   Rename "WhitelistWarningsGuard" to "AllowlistWarningsGuard"
*   Rename CompilerOptions#setCssRenamingWhitelist
    to #setCssRenamingSkiplist
*   Stop special-casing `goog.dom.TagName` for optimization now that existing
    passes can do the same thing.
*   Deleted deprecated CompilerOption `setIdeMode`.
*   Add iterable methods to URLSearchParams externs
*   Add polyfill for `TypedArray.fill`.
*   Add polyfill for `TypedArray.copyWithin`.

### June 28, 2020 (v20200628)

*   Simplified unused polyfill removal, with the result that more unnecessary
    polyfills may be present in the compiled output. This can happen when shares
    a name with an extern or structural type property.
*   Update AsyncGenerator type definition to have 3 template parameters instead
    of one. This brings it in line with TS and with the other Iterator types.
    Like those others, the 2 additional parameters aren't actually used yet.
*   `--shadow_variables` is now a no-op. The functionality scaled very poorly
    and proved to provide little meaningful value in practice.
*   Fix [crash with `new.target`](https://github.com/google/closure-compiler/issues/3607)
*   Fix [`instanceof` code removal bug](https://github.com/google/closure-compiler/issues/3618)
*   Add onChanged to Chrome extensions StorageArea
*   Rename WarningsGuard#Priority.SUPPRESS_BY_WHITELIST to SUPPRESS_BY_ALLOWLIST
*   Define TextEncoder.encodeInto API
*   Fixed
    `com.google.javascript.refactoring.RefactoringUtils.isInClosurizedFile` to
    recognize additional files declaring a goog.module as Closurized. Prior to
    this change, files that exported a class without requiring other imports may
    not have been recognized as Closurized.
*   Dropped 'duplicate requires' check from deps generator.
*   Addition of support for BigInt is in progress. With this release you may
    begin to see `bigint` mentioned in error messages. However, support for it
    isn't complete yet. We will announce when it is.
*   Declare HTMLVideoElement.requestVideoFrameCallback() and
    cancelVideoFrameCallback() See https://wicg.github.io/video-rvfc/

### June 14, 2020 (v20200614)

*   Fixed a bug causing `OptimizeParameters` to inline parameters for a case
    where the compiler couldn't actually see all calls to the function.
*   Avoid including polyfills for Promise and Symbol in `ES_2015` output. They
    were previously getting unnecessarily included due to their use in
    transpilation of async functions.
*   Fixed bug triggered by re-freezing an object frozen before the JSCompiler
    WeakMap polyfill loaded
*   Polyfill isolation mode no longer supports inserting
    frozen/sealed/non-extensible objects into the `Map`, `Set`, `WeakMap`, or
    `WeakSet` polyfills.
*   Add externs for all remaining Web Crypto APIs specified at
    https://www.w3.org/TR/WebCryptoAPI/ that weren't already present.
*   Support transpilation of classes that extend native classes (e.g. Map).
    Previously only extension of `Object` and various `Error` classes were
    supported for ES6 classes being transpiled to ES5.
*   Changed the default error format to include multiple lines from the original
    source.
*   Fix invalid code generation when attempting to convert RegExp constructors
    whose patterns start with * to regex literals
*   Fixed OptimizeParameters to stop inlining `yield` and `await` expressions
    into other functions.
*   Added the NO_OP conformance rule, useful for removing conformance
    requirements from a codebase without breaking downstream builds that extend
    them.
*   Fix bug in AngularJS externs where `templateUrl` field was not indicated to
    be in the prototype of `angular.$routeProvider.Params`.
*   Add jasmine externs for setSpecProperty and setSuiteProperty, due in v3.6.
*   Add jasmine externs to jasmine namespace for Spec.getFullName,
    Env.configure, SpecResult, JasmineDoneInfo, jsApiReporter, HtmlReporter,
    QueryString, HtmlSpecFilter, and Timer.
*   Add definitions for DOMPoint to externs
*   Adds missing externs for HTMLTextAreaElement: maxLength, minLength,
    textLength.
*   Add externs for maps api v3.41
*   checkGlobalThisLevel compiler option has been removed. Use the `globalThis`
    diagnostic group instead to trigger the global this checks.
*   Added a suggested fix for the new more accurate missing require warnings.
*   Report syntax error for usage of `yield` or `await` in default parameter
    values. This has always been illegal JS syntax. If this change seems to
    break your project, then it was really already broken and you should fix
    your code.
*   Fixed crash caused by using @export without depending on base.js

### May 17, 2020 (v20200517)

*   Change the externs to the current shape of [Trusted Types API](https://github.com/w3c/webappsec-trusted-types).
*   Convert `Symbol.iterator` to a library polyfill
*   Omit weak files from the bundle output.
*   Make count optional in Atomics. {notify,wake}
*   Start polyfilling `Symbol.iterator` for TypedArrays when they are present.
*   Remove the legacy_js_project flag.
*   Convert Symbol.asyncIterator to a library polyfill
*   Transpile `new.target` -> `this.constructor` in ES6 constructors.
*   Skip the JSC_UNKNOWN_EXPR warning for unknown expressions used as standalone
    statements.
*   Fixed `--isolate_polyfills` bugs causing runtime exceptions in compiler
    library code

### May 4, 2020 (v20200504)
*   Improved type inference of `===` operands when one operand is `?`
*   `--print_source_after_each_pass` sends files to
    `.../<debug_logdirectory>/JSCompiler/source_after_pass/...`
*   Add externs for `chrome.runtime.Manifest.prototype.icons`.
*   Add `Atomics.notify` API externs from ES2019.
*   Fix inappropriate return expression deduplication. Fixes
    [github issue 3462](https://github.com/google/closure-compiler/issues/3462)
*   Improve elimination of unnecessary polyfills only used behind guards, like
    `if (typeof Promise === 'function') { use(Promise); }`.
*   Add externs for `chrome.runtime.PlatformInfo`.

### April 26, 2020 (v20200426)

Note: A [problem](https://github.com/google/closure-compiler/issues/3587) was discovered
with this release after most of the work to publish it was already complete, so we did
another release the following week with that problem fixed and do not recommend using this
one.

*   Remove unused polyfills even if `--rewrite_polyfills=false` as
    long as polyfills are not force injected using options or flag
*   New optimization to replace Array.of calls with array literals (thanks
    @bugreportuser)
*   Fix `StorageEvent` API externs to match current W3C standard.
*   closure-compiler's AST now attaches inline trailing comments to the suitable parameters and
    arguments for function declarations and call-sites.
*   The compiler's runtime libraries now prefer the global `this` over `window`
    when searching for the global object to install polyfills on.
*   Stopped backing off inlining non-global variables whose names are "exported"
    according to the `CodingConvention` in use.
*   Add input support for numeric separators
*   `LogFile` (debug logging) now appends to existing files rather than overwriting them.
    All logs are cleared on compiler startup to prevent mixing across compilations.
*   Added WebGL 2.0 Compute externs to the set of standard browser externs

### April 6, 2020 (v20200406)
*    Conformance errors now print the conformance config file where the requirement was defined.
*    Remove extern definitions for obsolete static methods on Array that were only ever available on older versions of Mozilla products (such as FireFox and Rhino).
*    The "missing require" check no longer runs during lint.

### March 15, 2020 (v20200315)

*   Moved webassembly externs out of contrib.
*   Nullish coalesce operator (`??`) available with `--language_in=ES_NEXT`
*   Added a new suppression `useOfGoogProvide` to suppress the recently added
    lint error on `goog.provide`.
*   The "missing" and "extra" requires checks again run during lint/fixjs.
*   The "unknown class property" conformance check now recognizes ES6 classes.
    Previously a bug made it only apply to ES5 classes.
*   Fixed a bug that broke references to class static properties using the
    class's inner name. e.g.
    ```javascript
    const X = class InnerX {
      // gets collapsed to something like X$$staticMethod
      static staticMethod() {}
      method() {
        // left pointing at a nonexistent method
        InnerX.staticMethod();
      }
    }
    ```

### February 24, 2020 (v20200224)

*   Add externs for maps api v3.40
*   Avoid including polyfill for `globalThis` when input code doesn't use it.
*   Improved dead code elimination to realize exceptions thrown within try-catch
    blocks are side-effect free.
*   Fix optimization bug where functions using destructuring were sometimes
    incorrectly detected as side-effect-free and optimized away.
*   BANNED_PROPERTY (and related) conformance rules now apply to destructuring.
*   Extend "too many template parameters" warning to primitives like `string`
*   Drop support for modeling function hoisting in the typechecker. The
    typechecker expects all function references to be after the declaration. The
    only known behavioral change is that `goog.module` exports of ES5 classes
    that take advantage of function hoisting will no longer be typechecked. 
    This change does *not* affect the runtime behavior.
*   Report an error for `super` used in a normal function even when it's nested
    in a class method.
*   Report an error for `super()` calls in arrow functions within constructors.
    The JS spec allows them as long as `super()` is called exactly once before
    any references to `this` or `super.prop`, but they make it very hard to
    determine statically whether `super()` is being called correctly. Code that
    is calling `super()` from an arrow function is unnecessarily complex and
    should be refactored.
*   Deleted the deprecated diagnostic group `useOfGoogBase`
*   Bugfixes and improvements.

### February 4, 2020 (v20200204)

*   Warn on unrecognized types found in (spurious) enum and typedef template
    arguments
*   Linter now warns on @type annotations on getters and setters
*   Added optional expectationFailOutput parameter to toBe*() in Jasmine externs
*   Removed the .soy suffix from the <source> attribute in XMB files for
    generated code that wasn't generated by the Soy compiler.
*   Warn for probably misplaced @typedef on prototype and instance properties
*   Remove special handling of goog.base and deprecate the useOfGoogBase
    diagnostic. It was removed from Closure library in December 2019.
*   Fixed optimization bug where quoted properties in destructuring were
    sometimes renamed
*   Fixed type system bug that, in some cases, causes types to be dropped from
    unions
*   goog.addDependency (usually contained in deps.js files), is no longer
    supported as an alternative to goog.forwardDeclare. Projects should add
    explicit goog.forwardDeclare where necessary. NOTE:goog.forwardDeclare is
    deprecated in favor of goog.requireType.
*   Enforce that goog.module.declareLegacyNamespace(); is never passed any
    arguments.
*   Added SVG stroke properties to CSSProperties externs
*   Structural types are stringified with indentation.
*   Fixed spread argument transpilation bug where (/** @type {?} */
    (this.method))(...args) was rewritten to this.method.apply(null, args);

### January 12, 2020 (v20200112)
*   Added support for transpiling computed getters and setters in ES6 classes
*   `InlineAliases` is moved from after type checking to the beginning of
    optimizations. This may change the identifier names reported by the various
    post type checking check passes.
*   Presence of object rest now prevents removal of any of the other contents of
    the object pattern.
*   Stricten variable redeclaration checks to also report even if the declared
    types of the variable are a structural match.
*   Compiler-injected logic for finding the global object now works even when
    the output is wrapped in a strict-mode IIFE.
*   Allow `--browser_featureset_year=2020`
*   Add a `FeatureSet` describing the JS features implemented by all 2020
    browsers. Used when compiling with `--browser_featureset_year=2020`
*   Fix a crash when an interface would accidentally extend a union type.
*   Bug fix related to typing of properties on typedefs,
    e.g. `a.b` when `a` is also a `@typedef`

### January 1, 2020 (v20200101)
*   CrossChunkMethodMotion no longer moves methods containing `super`. Referring
    to `super` in a non-method function is a syntax error.
*   Fixed crash when using for-of without externs
*   Added polyfill for [globalThis](https://github.com/tc39/proposal-global).
*   Fixed bug where the `!` operator did not always remove null/undefined on
    `@typedef`s
*   Added missing `Symbol.iterator` on return of String.prototype.matchAll().
*   String.prototype.matchAll() now throws error when non-global RegExp is
    passed in as parameter. Updated to match spec updated in Oct 2019 TC39
    meeting.
*   Adds a warning for improperly formatted typeof annotations.
*   Implemented polyfill for
    [String.prototype.matchAll()](https://github.com/tc39/proposal-string-matchall).
    Only available if you specify `--lanauge_in=ES_NEXT`.
*   Added `globalThis` to externs. Polyfill not yet available - coming soon.
*   Added `String.prototype.matchAll()` to externs. Polyfill not yet available -
    coming soon.
*   Referencing `exports` before assigning to `exports` is now an error in a
    goog.module.
*   Add externs for maps api v3.38 and v3.39
*   Remove the deprecated `--dependency_mode STRICT`
    and `LOOSE`, which are synonyms for `PRUNE` and `PRUNE_LEGACY`,
    respectively.
*   Fix the type for `matchers.not` in the Jasmine extern.
*   Fix a bug where semicolons would be moved into a template string literal.
*   Removed obsolete diagnostic group 'es3'. Setting --language_in to ES5 or
    greater is the replacement for this diagnostic group.
*   Remove obsolete diagnostic groups 'newCheckTypes',
    'newCheckTypesCompatibility', and 'newCheckTypesExtraChecks'.
*   Remove obsolete diagnostic group 'ambiguousFunctionDecl'.
*   Deprecate and remove unuseful diagnotic group internetExplorerChecks. Setting the
    language_in to ECMASCRIPT5 or greater is the appropriate way to allow
    trailing commas in array or object literals.
*   Remove obsolete diagnostic group 'fileoverviewTags'
*   Fix an automatic semicolon insertion (ASI) bug with semicolons following
    newlines.

### November 11, 2019 (v20191111)
*   Enable a diagnostic against using bounded generics. The existing diagnostic had a logic error that effectively disabled it.
*   Deprecate unused diagnostic group ambiguousFunctionDecl.
*   Remove obsolete diagnostic group 'checkEventfulObjectDisposal'
*   Remove obsolete diagnostic group 'oldReportUnknownTypes'
*   Move missing goog.require and ES module import errors from the missingProvide diagnostic group to missingSourcesWarnings.
*   Enabled 'strictCheckTypes' with the '*' diagnostic group.
*   Warn for duplicate function parameter names when destructuring, using rest parameters, and default assignment.

### October 27, 2019 (v20191027)
*   Added externs for Jasmine 3.3.
*   Compiler enforces that `goog.require` is in the file scope, not within a
    function or conditional block.
*   Replace constant-inlined out-of-bounds array accesses with 'undefined'
    instead of reporting a warning
*   Fixed optimization bug on object destructuring declarations, which may cause
    a slight code size regression in some ES2015+ out targets.
*   In-browser transpiler is now build with J2CL rather than GWT.
*   Improved JSC_DUPLICATE_NAMESPACE error message to include the source of both
    the original provide and duplicate provide.
*   Remove support for duplicate sources in zip files.
*   Compiler now enforces that `goog.module.declareLegacyNamespace()` is
    immediately after the `goog.module` call.
*   Report undefined variable `goog` errors when using Closure dependency
    methods (`goog.{provide,require,module,etc.}`) without some definition of
    `goog`.
*   Removed "The template variable is unused" error to allow references to
    template variables from inline parameter declarations.
*   Added polyfill for `Promise.allSettled` (`ES_2020` feature).
*   Referencing the `exports` namespace of a `goog.module` through `this` will
    no longer work compiled in exported functions. (Note: this pattern already
    causes a `JSC_UNSAFE_THIS` warning and breaks uncompiled when using
    destructuring requires)
*   Back off from strict missing property errors while checking property absence
    (`if (x.y == null)` will no longer give a strict error if `x.y` is missing).

### September 29, 2019 (v20190929)
*   Added a linter warning for constant-by-convention names missing `@const`
*   It's now a compile error to not assign the result of a call to `goog.define`.
*   The GWT implementation of the debugger service has been deleted.
*   `CollapseProperty` now backs off on unsafe namespace aliasing.
    The warning is still emitted, but may be suppressed with `@suppress {partialAlias}`.
*   Allow `MSG_*` variables to be initialized as aliases of other `MSG_*` variables.
*   Improved the typechecker's ability to distinguish between different types with the same name. This may uncover new type errors.
*   Added `Promise.allSettled` (`ES_2020` feature) to standard externs definitions.
    Polyfill should be available in the next release.

### September 9, 2019 (v20190909)
*   CrossChunkMethodMotion now correctly recognizes methods declared in externs
    using ES6 class syntax. Previously it would ignore these, possibly causing
    incorrect movement of method definitions.
*   `!= null` and `== null` are now treated as property existence checks.
*   Fix for `@suppress` JSDoc on a property assignment causing access control
    checks to fail to warn.
*   Performance fix for outputting large files with source maps enabled
*   Subclasses of deprecated classes are no longer treated as deprecated
*   Fix bad code generation with async member functions:
    https://github.com/google/closure-compiler/issues/3217
*   Fix dropping duplicate symbols declarations as part of dependency pruning
    that prevented the same from being reported.

### August 19, 2019 (v20190819)
*   Fixed linter crash on default ES module exports
    (See https://github.com/google/closure-compiler/issues/3453).
*   Removed template parameter from `Argument` externs.
*   Remove blaze-out/bazel-out prefix from paths by default when checking
    conformance whitelists.
*   chrome_pass: remove assert, assertNotReached, and cr.ui.decorate from coding
    convention in favor of @closurePrimitive annotation in .js files directly.
*   Handle `{html: true}` option in `goog.getMsg`.
*   Correctly check property types when extending interfaces.
*   Compiler can preserve all source comments now including inline comments.

### July 29, 2019 (v20190729)
*   Allow @suppress annotations on compound assignment operators.
*   Correctly resolve forward declared types used as templated type arguements.
*   Optimize Array.concat by
    *   Replacing empty array literal from the front of concatenation with the
        first argument of concat function call
    *   Folding chained concat functions
*   Back off from collapsing property reads/writes when the property is declared
    in an or-expression or ternary-expression.
*   Fix `WeakMap` and `WeakSet` polyfills to treat non-object keys per the spec.

### July 09, 2019 (v20190709)
*   Improved support for incremental compilation of Polymer Behaviors. Properties defined by behaviors are now accessible to the elements that apply them.
*   Type checker no longer autoboxes primitives when validating case expressions in a switch statement.

### June 18, 2019 (v20190618)
*   Fix for misassociating property declarations that resulted in access control
    false positives.
*   Fix ES modules referencing CommonJS modules.
*   Allow js_library checking of untranspilable ES2018 features.
*   Fixed false negatives on invalid goog.require calls in goog.provide files
*   Remove pure undefined default values in parameters and object patterns.
*   Make local enums and typedef types non-nullable by default, to be consistent
    with the default nullability of global types.

### May 28, 2019 (v20190528)
*   `--devirtualize_prototype_methods` is now `--devirtualize_methods` due to
    added support for static methods. The old name is still temporarily
    supported for transitioning.
*   Abstract classes no longer need explicit abstract method overrides to
    implement interfaces.
*   Fixed a crash in chrome_pass with defineProperty.
*   Introduce a new compile time error if an ES6 class is passed as the first
    argument to `goog.inherits`, use the `extends` keyword instead.

### May 13, 2019 (v20190513)
* Flag changes:
    * Introduced a new flag `--browser_featureset_year` to control the `goog.FEATURESET_YEAR` define and to set a default `--language_out`.
    * Removed deprecated `--new_type_inf` and `--allow_method_call_decomposing` flags from the command line runner.
    * When using `--dependency_mode=PRUNE_LEGACY` or `--manage_closure_dependencies`, a file passed under `--weakdep` can no longer be an entry point.
* Added typing support for `this` and `super` in static methods:
    * `this` and `super` are now typechecked in methods declared with the `static` keyword. Static methods declared using other syntaxes are inferred to be free functions.
    * For functions whose `@this` type is `*` or `undefined`, typechecking permits free calls.
* Improvements to unused code removal:
    * Unused polyfills are now removed by RemoveUnusedCode, which should give smaller output code size in some cases.
    * Unused parameters of computed setters are no longer removed.
    * Unused parameters are retained in computed-property setter declarations.
* Typechecking verfies that classes declare the instance properties of their implemented interfaces (e.g. properties declared in the "`constructor`" of the interface).
* Assigning a value to an `@typedef` simple name is only allowed if the value is a literal or cast and the name is `const` or `@const`.
* Aliases are now followed when determining whether the initializer for `goog.define`s and `@define`s is valid (i.e. a compile-time constant).
* Fixed incorrect printing of comma expressions used as computed property keys or values, which led to syntax errors.

### April 15, 2019 (v20190415)
*   Fixed a NullPointerException that occurred when using `this` in a default
    parameter value for an async function or async generator.
*   Static methods are now checked as overrides, as per class-side inheritance.
    This only applied to ES6 classes.
    *   The type of a static method is now checked to be compatible with any
        superclass definitions
    *   `@override` annotations are validated on statics.
*   Removal of various obsolete ES5-era or transitional Java APIs:
    *   SyntacticScopeCreator
    *   MarkNoSideEffects
    *   NodeTraversal's traverseEs6ScopeRoot, traverseTyped, and traverseRootsTyped
*   In the GWT version of the parser, Greek Capital Letter Delta is now accept
    in a identifier.
*   For users of the Java API: TypeCheck#processForTesting enforces its inputs
    are ROOT nodes.
*   Function aliasing, including class constructors and `super()`, is now
    considered when analyzing function purity.
*   Type checking performance improvements

### March 25, 2019 (v20190325)
*   Removed some rewriting of the input JS pre-typechecking. User-visible results include:
    *   Names in error messages match the original JS better
    *   The compiler is stricter about requiring namespaces to be const object
        literals.
*   Iterable spread is now preserved for side-effects.
*   Some type references not explicitly annotated with `!` are now considered nullable. Code should always explicitly specify nullability.
*   Fixed bug when dropping const-ness and JSDoc on names exported from `goog.module`s.
*   Improved type checking of ES module transitive imports / mutable exports.
*   `export * from` within ES modules is now supported.
*   `--shadow_variables` is now disabled by default. It does not scale well and provides little value in most cases.
*   Type checking performance improvement.
*   Added polyfills for trimStart/trimEnd.

### March 1, 2019 (v20190301)
*   No longer artificially require ES6 externs when transpiling down to ES5.  
    If type checking and transpilation are both on and standard externs are missing, 
    it's possible to see some type warnings from internal runtime library code.
*   The `--allow_method_call_decomposing` flag has been removed as the
    functionality is always enabled.
*   Fixed a bug with printing of spaces in template literals.
*   Changed the API of Result.errors and Result.warnings from `JSError[]` to
    `ImmutableList<JSError>`.
*   Simplified the API for `CodingConvention.AssertionFunctionSpec`
*   Compiler will no longer skip passes that don't understand newer features
    in the source code that aren't being transpiled away. Instead it will halt
    with an error message.
*   `ECMASCRIPT_2019` is now available for use as `--language_in`.
    See features added at
    https://github.com/tc39/proposals/blob/master/finished-proposals.md
*   Default `--language_in` is now equivalent to `ECMASCRIPT_2018`.
*   `ECMASCRIPT6`, `ECMASCRIPT_2016`, and `ECMASCRIPT_2017` are now fully
    supported for `--language_out`.
    JSCompiler team has confirmed that resulting output is no bigger than
    it would be for `ECMASCRIPT5`. Generally it is smaller.

### February 15, 2019 (v20190215)
*   Prerequisite changes to improve support for goog.define in modules.
*   Fix crashes with language_out=ES2017.
*   "--allow_method_call_decomposing" is now enabled by default.
*   ES_2019 feature: unescaped unicode line and paragraph separators in string
    literals are now supported https://github.com/tc39/proposal-json-superset
*   It is now an error to request a compiler pass that does not support all
    language features present when the pass runs (i.e. if necessary
    transpilations have not also been requested). The pass will still be run.
*   Polyfill for `Math.hypot` now works correctly for zero or one argument.
*   Blank lines are now preserved in template literals
*   ES2018 is now type checked when language_out=ES2018 is specified.
*   Declaring a class constructor using `get`, `set, `async`, or `*` is now a
    parse error.
*   The BanUnknownDirectThisPropsReferences conformance check now allows
    references to properties that are explicitly declared to have a type of `?`
    (Unknown).

### January 21, 2019 (v20190121)
*   Report a warning if the left-hand side of a logical operator is guaranteed
    to always have the same boolean value.
*   Devirtualization of prototype properties optimization is now enabled for
    ES6 classes, even when not transpiled to ES5.
*   Type inference of `Promise.prototype.catch()` was corrected so that the
    return value is `Promise<X>` when the callback's return value is
    `Thenable<X>`. It previously was incorrectly inferred as
    `Promise<Thenable<X>>`.
*   Missing properties on functions are reported more aggressively with
    "missingProperties" as they are with "strictMissingProperties".  This
    also improve type checking performance on larger projects.
*   Property collapsing is more aggressive on ES6 classes in untranspiled mode.
*   Improved type checking for object spread. e.g. `const o = {a:1,
    ...otherObj};`
*   `--record_function_information` flag is no longer supported.
*   Union types of arguments are decomposed when matching function template
    types, allowing more matches. Example, passing `Array<string>|Set<string>`
    to a function expecting `Iterable<T>` now infers `T` to be `string`.
*   Runtime type checks can be added to ES6 code.

### January 6, 2019 (v20190106)
*   Enabled the `--inline_properties` optimization for ES2015-ES2017 output.
*   Improved dead code removal for empty destructuring patterns.
*   `@noinline` now prevents constant parameter values from being inlined into
    the body of functions.
*   Improved property collapsing for destructured objects in ES2015 output

### December 10, 2018 (v20181210)
* Enable the `--ambiguate_properties` optimization for ES2015-ES2017 output.
* Enable processing of `@define` annotations in checks-only mode, so errors related to them will be reported.
* Report warnings when calling a superclass abstract method with `super.foo()`.
* Fix crash in PolymerPass when missing externs for PolymerElement.

### November 25, 2018 (v20181125)
* Improved typechecking for [async/await](https://github.com/google/closure-compiler/issues/2540)
  and ES6 class constructor calls
* Typechecking now available for builds with `--language_out=ECMASCRIPT_2017`.
  Previously it was disabled.
* `--disambiguate_properties` and `--replace_strings` work for builds with
  `--language_out=ECMASCRIPT_2017`.
* Subtyping is now recognized for recursive templated types (e.g. `Child extends
  Parent<Child>`)
* Access-controls are now applied to property accesses via destructuring.
* Report errors for comparisons involving `Number.NaN`, since they are always
  `false`.
* Fixed [crash](https://github.com/google/closure-compiler/issues/3145) on
  computed properties in destructuring
* Correct transpilation of ES6 class getters and setters with quoted property
  names.
* Added [polyfill for Math.fround](https://github.com/google/closure-compiler/issues/3141)
* Warnings about JSDoc annotations in non-JSDoc comments (e.g. `/* @private */`)
  have been moved to the linter and will no longer be reported during builds.
* Linter now gives suggested fixes for reference types without ! or ?
* Fixed crash when multi-line template literals are present towards the end
  of the file.

### October 28, 2018 (v20181028)
*   Typechecking will now run for builds with `--language_out=ECMASCRIPT_2015`.
    Previously, typechecking was disabled for output level better than `ES5`.
*   Made name alias inlining more aggressive to fix some cases where property collapsing causes bad output
*   Change async transpilation to work around an MS Edge bug.
    https://github.com/google/closure-compiler/issues/3101
*   Fixed bug causing some properties aliasing global names to be set to `null`
*   Removed obsolete `RemoveSuperMethods` pass.
*   Improved type checking for destructured parameters and assignments.

### October 8, 2018 (v20181008)
*   Typechecking and other checks now see untranspiled classes and arrow functions
*   ES2018 feature: Allow previously invalid escape sequences in tagged template
  literals, according to https://tc39.github.io/proposal-template-literal-revision/.
*   The "Symbol" polyfill is now injected less often.
  Specifically, the use of "for-of" no longer requires the Symbol polyfill.
*   GitHub issue #3080: The WeakMap polyfill no longer causes infinite recursion
  when freezing objects recursively.
*   Rename the helper method for declaring a closure namespace from an
  ES6 module from from goog.module.declareNamespace to goog.declareModuleId.
*   PolymerPass now generates goog.exportProperty calls (instead of externs)
  when protecting Polymer element methods from renaming and dead code removal.
*   Fixed crash when printing "typeof" in type expressions when printing JSDoc.
*   We no longer accept missing template parameters as unknowns in the unknown
  conformance checks.
*   Fixed bug using quotes from command line on windows. (GitHub issue #3081)
*   Recognize named capture groups in Regex expressions and report error when
    trying to transpile named capture groups
*   `@suppress` is now legal on computed property methods and applies to both
    the property name expression and the method body.
*   `symbol` is now an allowed type in `in` expressions
    [(github issue 3060)](https://github.com/google/closure-compiler/issues/3060)

### September 10, 2018 (v20180910)
*   The "typeof" type expressions now have preliminary support. Type expressions
    of the form "typeof x" can be used to refer to types of objects that are
    were previously anonymous such as a namespace, an enum definition or a
    constructor.
*   Improved the "duplicate provide" error message to include the other file.
*   Fixed bug where compiler failed to warn on variables referenced before
    declaration inside inner block scopes.
*   (`ES_2018`) Recognize unicode property escapes in regular expression
    literals. Generate a warning since we don't transpile those yet.
*   (`ES2018`) Recognize lookbehind assertions in regular expression literals.
    Generate a warning since we don't transipile those yet.
*   `@protected` properties of outer classes are now accessible from inner ones.
*   Fix for bad code do to inlining of references to "super".
*   Applied Java 10 C2 bug workaround.
*   Initial implementation for async generators available with an `ES2018` input
    language level.

### August 5, 2018 (v20180805)
* GWT version of the compiler now supports `--js` and `--externs` flags when
  executed by NodeJS.
* Fixed a crash that occurred when extending a non-builtin class named `Object`.
* Fixed a bug causing incorrect escaping in template literals for `ES6+`
  output.
* You can now specify `STABLE` for input and/or output language level to request
  the latest version of JavaScript that is fully supported by the compiler.
  Currently, this means `ES_2017` for input and `ES5` for output.
* Fixed bug in [optimization of shorthand assignments](https://github.com/google/closure-compiler/issues/3017) (e.g. `+=`, `*=`)

### July 16, 2018 (v20180716)
* Add a pom file for building a RefasterJs jar.
* Type checker now knows that global `let` and `const` declarations do not add properties to the global object. To avoid a missing property error, you must use `var foo` if you want to later access `window.foo`.
* Improvements to CommonJS rewriting support.
* Improvements to the typechecking of several ES6 patterns, including rest/spread.
* Forward references to types now preserve their type arguments.
* The npm distribution now includes native binaries for MacOS and Linux. Native binaries offer faster compile times without requiring a JVM.

### June 10, 2018 (v20180610)
* This release now contains the JS version of Closure Compiler inside the standard NPM package.
* Consider @desc as @const
* Remove special handling for @interface initialized to goog.abstractMethod
* Fix inline type annotations of optional and var_args parameters.
* Improve CommonJS UMD pattern detection.
* Removed special handling for `Error` and subclasses: they will no longer be
  implicitly added if missing from (nonstandard) externs.
* Typechecking and other checks now see untranspiled template literals,
  computed properties and object literal member functions.
* Fixed bug where the compiler wasn't reporting duplicate destructured
  declarations.
* Improved type inference for 'assign op's such as `*=`
* Typechecking and other checks now see untranspiled for-of loops
* Fixed [incorrect code removal
  bug](https://github.com/google/closure-compiler/issues/2924). The fix makes
  an optimization more conservative so may slightly increase code size.
* Compiler preserves an input source map's `sourcesContent` in the output
  source map
* Renamed code-splitting flags from "module" to "chunk".
  e.g. cross_module_code_motion -> cross_chunk_code_motion
* Removed JSDoc parser support for "namespace types" (e.g. `{jQuery.}`)
* Overhauled type inference's data structures for tracking flow-sensitive types
* Typechecking and other checks now see untranspiled let/const variables.
* Fixed bug in [for loop transpilation in
  generator functions](https://github.com/google/closure-compiler/issues/2910)
* Fix scoping of catch variables during type inference.

### May 06, 2018 (v20180506)
- Simplified transpilation of async functions for decreased code size
- Fixed [redeclared variable error for rest destructuring](https://github.com/google/closure-compiler/issues/2909)
- [Corrected handling of +0 and -0 in Map and Set polyfills](https://github.com/google/closure-compiler/pull/2905)
- `--new_type_inf` is now a no-op flag. NTI has been turned down (consider `strictCheckTypes` as an alternative).
- Removed the deprecated `Text.prototype.replaceWholeText` API
- Fixed bug in code generator for arrow function bodies with object literals
- Implemented [string literal trim() folding](https://github.com/google/closure-compiler/pull/2900)
- Updates to support for local (non-global) @typedef declarations
- Updates to improved support for local (non-global) type declarations.
- Modified handling of extern declarations such that a class' @implements are fulfilled implicitly rather than always requiring explicit redefinitions.
- Improved type inference on callbacks without an explicit type declaration
- Better error checking in generator functions. Type checking code now understands generators and sees them un-transpiled for all use cases.
- Better error checking for let/const and for-of loops in `--checksOnly` mode only. Type checking code sees them as untranspiled.

### Apr 02, 2018 (v20180402)
* Fixed flaky stack overflow problem in serialization code.
* Corrected corner-case behavior of `Array.prototype.includes` polyfill.
* Add a polyfill for `Promise.prototype.finally`.
* Introduce "Global" as the type of the global object.
* Compiler will now keep running compiler passes after hitting non-fatal errors (i.e. those that are not errors by default).
* Misplaced `@suppress` warnings are now on by default.

### Mar 19, 2018 (v20180319)
* Added "misplacedSuppress" diagnostic group for warning about misplaced
  @suppress annotations.
* "strictMissingProperties" is now stricter about properties referenced in
    conditionals.  Only "typeof" is allowed as a property existence check.
* Fixed a [bug that prevented transpilation of `**`](https://github.com/google/closure-compiler/issues/2821) in some cases.
* Fixed an [unused code removal bug](https://github.com/google/closure-compiler/issues/2822)
* Improved transpilation of spread `...expr` to improve code size and prevent
  a code pattern that blocked removal of unused code.
* Stricter "strictMissingProperty" checks.
* "`strictCheckTypes`" diagnostic group to enable both the "strict operators"
  and "strict property access" checks
* Added preliminary support for 'symbol' as a primitive type to the type system.
* Bugfixes/performance improvements in the `--checks_only` mode
* ES6-output improvements:
    * `StripCode` now supported
    * `ReplaceMessages` now supported
* Added new "strictPrimitiveOperators" diagnostic group for getting stricter
  NTI-like checking of primitive operators.

### Feb 04, 2018 (v20180204)
*   Replace `RemoveUnusedClassProperties` and `smartNameRemoval` passes with
    new `RemoveUnusedCode` optimization pass.
*   Introduce "strictMissingProperties" that emits warnings when properties are
    referenced and can only be found on subtypes.  These warnings can be
    suppressed with "missingProperties" or "strictMissingProperties" but are
    enabled with "strictMissingProperties".
*   Now allow @suppress JSDoc to be attached to any type of declaration
    (including let/const/class) instead of only functions/methods.
*   Add support for transpiling object rest and spread.
*   Removed obsolete optimization pass `ChainCalls`
*   Deleted the legacy option to optimize the property registry in OTI.
*   Block-scoped function declarations are now considered an ES6 feature and are
    treated as lexically scoped to the declaring block (per ES6 strict
    specification) and are no longer accessible elsewhere outside that block.
*   Fix polyfill bug where `Math.hypot(0, 0)` was returning `NaN`.
*   Add support for mutable exports and forbid duplicate exports in ES6 modules.
*   Misc bugfixes and improvements.

### January 01, 2018 (v20180101)
* If you are using the inlineFunctions or inlineLocalFunctions boolean fields of the compiler options, you will have to migrate to the getter/setter methods setInlineFunctions(Reach) or getInlineFunctionsLevel() instead.
* Experimental support for @noinline annotation. It will prevent inlining of symbols by the inlineVariables and inlineFunctions passes, but not other inlining passes.
* Compiler.addToDebugLog is deprecated. If you have a custom pass that calls it, please switch to logging the information with a standard Java Logger.
* Labeled function and class declarations are now a parse error. These are illegal in ES6 strict mode.
* ES6-output improvements:
  * RescopeGlobalSymbols now supported
  * ReplaceIdGenerators now supported
  * ClosureCodeRemoval now supported
  * CrossModuleMethodMotion now supported
* Made AggressiveInlineAliases more aggressive about inlining constructors to prevent decomposition from hiding references to static properties that CollapseProperties needs to replace.
* Improved optimizations of modules in SIMPLE mode with wrapped output.
* Fixed a bug that prevented property names defined in extern JSDoc from being recognized when type checking was disabled. This led to extern properties being renamed in some cases.
* RemoveUnusedPrototypeProperties pass removed and its functionality merged into RemoveUnusedCode. This gives a performance boost by reducing redundant work.

### December 03, 2017 (v20171203)
* Fixed a bug that broke the JavaScript (GWT) version of the compiler (closure-compiler-js).
  That project and its npm have now been updated.
* Fixed a bug causing incorrect code removal.
  https://github.com/google/closure-compiler/issues/2365
* ES6-output improvements:
  * J2CL passes now supported
  * `CrossModuleCodeMotion` now supported
  * `name(Un)MappedAnonymousFunctions` now supported
  * OptimizeCalls now supported
* Several improvements to handling of and interoperation between CommonJS and ES6 modules.

### November 12, 2017 (v20171112)
* For projects invoking the compiler from Java code: Removed Compiler method reportCodeChange. Code that was using this should switch to reportChangeToChangeScope or reportChangeToEnclosingScope.
* Improvements to optimization passes related to OptimizeCalls.

### October 23, 2017 (v20171023)
* When using the compiler to manage dependencies, made the dependency ordering match that of ES6 modules. In particular, this means that dependencies are processed depth first in the order of the imports in the file, starting from the entry points. For more details, see https://github.com/google/closure-compiler/pull/2641
* "use types for optimizations" no longer triggers optimizations of comparisons with null and undefined.

### September 10, 2017 (v20170910)
* Improvements to performance of optimization passes.
* Improved support for goog.modules with object literal exports style (e.g. exports = {Foo, Bar})
* Disallow 'is' property in a Polymer behavior.
* Continuing improvements to experimental ES6 output mode.
* Building the compiler and its tests now requires Java 8.

### August 6, 2017 (v20170806)
*   Several improvements to optimization of output modes > ES5.
    Type checking and optimizations that depend on that don't work yet, though.
*   `CrossModuleCodeMotion` now moves all symbols to the current best possible
    module in a single run of the pass, greatly reducing execution time for this
    pass. It also does a much better job of selecting the best module, leading
    to significant transitive module size improvement.
*   ES6 static method inheritance improved where possible.
    ES6 classes are supposed to inherit static methods by prototype inheritance
    between the 2 constructor functions. This is now implemented for all browsers
    that can support it, falling back to copying properties on those that don't.
*   `ECMASCRIPT_2017` is now the default language mode.
*   The `.throw()` method for a generator now works correctly when its argument
    is undefined. Previously it behaved as if `.next()` had been called.
    This also fixed a bug with async functions that caused promises rejected
    with `undefined` to be treated as if they were completed successfully.
*   The output language defaults to ES5 instead of to the input language.
*   Promoted a couple of useful diagnostics to their own diagnostic group:
    -   `jsdocMissingType`: Uses of `@param`/`@return` without a type (was lint-only).
    -   `unnecessaryEscape`: Backslashes in string literals that are ignored
        (was lint-only).
    -   `misplacedMsgAnnotation`: Warnings about misplaced `@desc`/`@hidden`/`@meaning`
        JSDoc annotations (was only available as part of `misplacedTypeAnnotation`).

### June 26, 2017 (v20170626)
* `--language_out` now allows ES2015 and higher. Passes which don't yet know
  how to handle ES2015 (including typechecking, and most optimization passes)
  will be skipped, so this is not recommended for production builds.
* Several passes updated to work with ES2015+ code.
* Read access to super class properties via `super.someProperty` is now
  allowed.
* Fixed a bug that broke access to `this` within an async arrow function
  nested in another async function.
* Changed the default ES6 module resolution mode to match that used in
  browsers.
* Tools that call the compiler via its Java API and provide their own PassConfig
  may need to include a featureSet() method in each PassFactory.
* CrossModuleCodeMotion now always picks the module with the smallest number
  of dependents when moving definitions. This will reduce the amount of code
  that must be loaded for some modules.
* Now ES6 language mode defaults to strict mode. If you don't want
  strict mode checks, use the --strict_mode_input=false flag.
* Allow --strict_mode_input flag to work in all language modes, including ES3.

### May 21, 2017 (v20170521)
* Fixed a bug that caused the polyfill versions of Promise.all() and Promise.race() to be renamed when disambiguate properties was enabled.
* Fixed a bug in CrossModuleCodeMotion that caused some global variable definitions to become pinned to a module, when they should be allowed to move.
* Allow --emit_use_strict flag to work in all language modes, including ES3.
* NTI: handle record/interface properties declared on THIS inside the constructor.
* Make Reflect.construct polyfill compatible with ES3.
* New BANNED_NAME_CALL conformance check

### April 23, 2017 (v20170423)
 * In RescopeGlobalSymbols, handle the case where a variable is defined both in the externs and in the source.
 * Symbol polyfill uses quoted properties to access members of $jscomp.global.
 * NTI: improve typing of .call/.apply on methods of generic classes.
 * Support compilation of programs with missing definitions.
 * Absolute path imports now resolve when combined with `--js_module_root` flags.



### April 9, 2017 (v20170409)
 * Added diagnostic group `tooManyTypeParams` to check for invalid templatized types such as `Array<string, number>`.
 * Use a size heuristic to stop the optimizations early when they are not making enough changes. Improves compile time by about 10%.
 * Added BanCreateDom conformance check.
 * Fixed a bug preventing definition of `@record` properties in the constructor when collapse properties is enabled.
 * Bugfix related to unions of unresolved types in the old type checker.
 * NTI: improve type checking of unannotated callbacks.
 * NTI: handle `.call` and `.apply` during @const inference.
 * The call to `goog.module()` must be the first statement in a goog.module. For instance, if you had `goog.setTestOnly()` before `goog.module()` the order must be reversed.
 * Check for temporal dead zone violation in `for (let x of [x]);`
 * Don't warn when an alias inside a goog.module is only used in type annotations.
 * Performance improvements to the new type inference and to InlineSimpleMethods.

### February 18, 2017 (v20170218)
 * --polymer_pass flag is deprecated in anticipation of Polymer 2 support. Use --polymer_version=1 instead.
 * Add a `--module_resolution` flag to specify which algorithm the compiler should use to lookup modules. The flag has 3 options:
   + `LEGACY` (default) - the same behavior the compiler has used historically. Users are recommended to begin migrating away from this behavior.
     * Module paths which do not begin with a "." or "/" character are assumed to be relative to the compilation root.
     * Modules files can only have ".js" file extensions. The extension is auto-added if the import omits them.
   + `NODE` - uses the node module resolution algorithm.
     * Modules which do not begin with a "." or "/" character are looked up from the appropriate node_modules folder. Includes the ability to require directories and .json files.
     * Files may be any extension. The compiler searches for an exact match, then ".js", then ".json". `package.json` files should be supplied as source to the compiler.
   + `BROWSER` - mimics the behavior of native module loading in browsers (browser support for this is close, but not yet generally available). The behavior of this mode will be adjusted to match the behavior of browsers as they release.
     * Modules must begin with a "." or "/" character.
     * Modules import statements must include the file extension.

### January 24, 2017 (v20170124)
* Improved optimization of modules with named exports.
* Several small improvements to compile-time performance.
* Add a check that `super()` is called before accessing `this`.
* Make type-based optimizations safer about null/undefined.
* Improved handling of parameterized object types in NTI (`Object`).
* In NTI, allow any `IArrayLike` to be used with `.apply` function calls.

### December 1, 2016 (v20161201)
* New type inference supports `@abstract`.
* CommonJS modules now use the node module resolution algorithm and support using JSON files as module source.
* Bugfixes in DisambiguateProperties for interface inheritance and implicit use of interfaces.

### October 24, 2016 (v20161024)
* Polyfills for Promises.
* Support for async/await by transpiling to generators.
* Improvements to CommonJS module rewriting.
* New flag for strict-mode output: --emit_use_strict.
* Duplicate keys in an object literal are always an error, even if strict mode is not explicitly enabled.

### September 11, 2016 (v20160911)
* Various compile time performance improvements to the optimization passes.
* Allow applying input source maps to the generated source map
* Better support/fixes for @abstract classes and methods
* Add externs for jQuery 1.12 & 2.2
* Add Boolean.prototype.valueOf to externs
* Externs for Google Maps API 3.26

### August 22, 2016 (v20160822)
*   Improved support for refactoring arrow functions containing 'this' or
    'arguments'
*   Improve branch coverage instrumentation
*   Improvements for NTI compatibility mode
    *   Don't warn when passing a method where a function is expected
    *   type variables in body of a function are unknown.
*   Improvements for NTI
    *   Recognize namespace aliasing in goog.module exports
    *   Don't mistake specialized versions of inherited properties for own
        properties. (Fixes #1943 on Github.)
*   Added externs for fetchapi Headers: keys, values, entries, based on:
    https://developer.mozilla.org/en-US/docs/Web/API/Headers
*   Add a distinction between `@const` and `@final`. Only @final means a class
    or constructor is not subclassible.
*   Warn if an ES5 class extends an ES6 class.
*   The exponentiation operator `**` is now supported for --language_in=ECMASCRIPT7
*   More compile-time checks for @abstract classes and methods
*   New optimization: RemoveSuperMethods, which deletes methods that only make
    a super call with no change in arguments

### July 13, 2016 (v20160713)
* Internal change to the way the AST is represented, which results in much
  faster compilation, especially for projects with large files. (https://github.com/google/closure-compiler/commit/90ac054be8d735c5b5530cf87923bd3d99697335)
* The jar downloaded from dl.google.com now includes the version
  number in its filename (`closure-compiler-v20160713.jar`)

### June 21, 2016 (v20160619)
* Improved module rewriting for destructured imports.
* New --checks_only flag that causes the compiler to not run any optimizations.
* Allow prevention of property renaming based on the coding convention.
* Removed support for the `var name = goog.require(...).name;` import pattern. The suggested alternative is `const {name} = goog.require(...);`
* Added support for @abstract on methods as an ES6-compatible alternative to goog.abstractMethod.
* Type based renaming now supports Object.defineProperties and thus ES6 getters and setters.

### May 17, 2016 (v20160517)
* ES6 library polyfills (e.g. `Map`, etc) are on by default. They
  can be disabled with `--rewrite_polyfills=false`.
* Improve the rewriting of goog.modules. As a result, we now disallow accessing
  (non-legacy) goog.modules by their fully qualified name. Also, includes a few
  new checks of goog.module misuse.
* Allow destructuring imports inside goog.module
  (e.g. `const {assert} = goog.require('goog.asserts');)
* Switch to ES6 module ordering
* Fix parsing of very deeply nested binary operators to avoid stack overflows.
* Check for missing/incorrect usages of ES6 super statement.
* Remove the "unnnecessary cast" check.
* Modules which are imported, but have no exports are now properly rewritten
* Updated dead assignment elimination to eliminate assignments in var/let/const
  statements, saving a few hundred bytes for some projects.
* Better optimization of switch statements that contain a default case.
* More aggressive constant folding.
* CommonJS module processing now rewrites `require.ensure` calls.
* Command line flag output is now grouped by category for greater readability
* Module processing now properly recognizes absolute paths

### March 15, 2016 (v20160315)
* Improved code removal of Object.defineProperties calls
* Compiler support for goog.reflect.cache, which allows functions with
  internal caches to be considered side-effect free for optimizations.
* Linter now finds missing/extra ES6 import statements.
* The `inferredConstCheck` diagnostic group is a no-op. Use the
  `com.google.javascript.jscomp.ConformanceRules$InferredConstCheck`
  conformance check instead.
* The new type inference handles IObject/IArrayLike.
* Run FlowSensitiveInlineVariables before function inlining to improve code
  size.
* `--use_types_for_optimization true` now enables additional constant folding
  opportunities.
* Allow entry points that don't provide any symbols (like ES6 or CommonJS)
* CommonJS module rewriting is now only triggered by the presence of an export.
    Previously `require` calls also triggered a rewrite.
* New lint checks:
  - Duplicate case statements in a switch
  - Check for the `for (!x in obj)` pattern, which is usually a mistake
  - Check for missing @return JSDoc

### February 08, 2016 (v20160208)
* Fixes the bug about file globs that was in the rolled-back January release.
* Reorganization of the dependency management flags. `--manage_closure_dependencies` and
   `--only_closure_dependencies` are deprecated and replaced by `--dependency_mode`.
   See https://github.com/google/closure-compiler/wiki/Managing-Dependencies 
* CommonJS Modules now interoperate with ES6 modules with improved type inference.
    See https://github.com/google/closure-compiler/wiki/JS-Modules
* Improved handling of templated types for IObject and Array to make it
  possible for Array to implement IArrayLike.
* Removed the ability to use `,` for union types as it led to confusing
  situations. You will need to rewrite types like `{number,string}` as
  `{number|string}` and you may find some new type errors because types such as
  `Object<number|string, X>` were previously not parsed as intended.
* Minor improvements to constant folding. For example, "new Date == null"
  becomes "false", Boolean(x) to !!x
* Allow inlining of uncollapsed constructor properties if type based
  optimization are enabled and the value is immutable.
* The coverage instrumentation pass will no longer crash on arrow functions.
* New flag for the command-line runner: assume_function_wrapper.
  This flag is used to indicate that "global" declaration will not actually
  be global but instead isolated to the compilation unit.
* Passing a "..." to a constructor should now work correctly. The output
  involves Function.prototype.bind so you'll need to set --language_out=ES5 to
  make this work.
* Improved exit minimization in switch statements
* The inferredConstCheck diagnostic group is deprecated, and will soon be
  replaced by the
  `com.google.javascript.jscomp.ConformanceRules$InferredConstCheck`
  conformance check.
* New lint check: Warn about useless blocks like
```javascript
return
    {foo: 'bar'}; // Useless block
```
or
```javascript
if (denied) {
  showAccessDenied();
} { // Useless block
  grantAccess();
}
```

### January 25, 2016 (v20160125) [rolled-back]

### December 16, 2015 (v20151216)
* Allow input and output as JSON streams to support the gulp and
  grunt workflows. See: <https://github.com/ChadKillingsworth/closure-compiler-npm/blob/master/README.md>
* Compiler web service and CommandLineRunner now default to
  transpiling from ES6-ES3 (instead of assuming input is ES3).
* Pretty print record types in error messages to make it easier
  to find the source of type mismatches.
* HEURISTIC renaming has been removed.
* Don't warn if the root of a namespace is defined twice in the externs.
* Some optimization passes now understand Object.defineProperties

### October 15, 2015 (v20151015)
*   ECMAScript 6 is officially supported.
*   CollapseProperties will now collapse in more cases, possibly resulting
    in fewer warnings and smaller code size.
*   Destructuring is supported for goog.require()s in ES6 modules.
*  ` @struct` is now supported for `@interface` and `@record` declarations in
    the "old" type inference (The NTI has this behavior by default).
    `goog.defineClass` `@interface`s are `@struct` by default (like
    `@constructor`)
*   "unknown property" conformance check now checks `@interface` and `@record`
    types.
*   Improved performance for parsing arrays, objects, and
    parenthesized expressions.
*   Typechecking improvements affecting module code.
*   Inline type annotations on function parameters with `@type` will now be
    respected.

### September 20, 2015 (v20150920)
* Better support for IArrayLike, which checks usages of []
* Improved handling for arguments array
* Support structural typing for interfaces with @record.

### September 1, 2015 (v20150901)
* Support for different externs sets via the --env flag.
    + `--env=CUSTOM` replaces the `--use_only_custom_externs` flag.
    + Core language externs are always loaded - even for custom environments.
* Improved handling of type aliases, including types exported across modules.
* Use ES6 module ordering when in ES6 mode.
* Remove checkStructDictInheritance diagnostic group (it no longer did anything)

### July 29, 2015 (v20150729)
* Aggressive variable checks are now on by default, and the setAggressiveVarCheck flag has become a no-op.
* Duplicate license comments will be omitted in the output.
* Stringifiable object keys check is on by default.
* Removed double-bar syntax (||) for declaring unions in type annotations.
* Removed --accept_const_keyword flag. Use --language_in=ES6_STRICT instead.
* --jscomp_(warning|error|off) flags now accept the '*' char as a wildcard.

### June 9, 2015 (v20150609)
*   AMBIGUOUS\_FUNCTION\_DECL is now an error by default.
*   New lint check: Extended interfaces should be goog.require'd
*   --generate_exports generates goog.exportProperty calls instead
    of goog.exportSymbol for @exports on prototype properties.
*   Several fixes and improvements to the [[new type inference|Using-NTI-(new-type-inference)]]
*   Several improvements to the PolymerPass.
*   ES6-related fixes:
    * Fix bug with redeclared this for arrow functions
    * Fix goog.scope rewriting to work in ES6 mode

### May 5, 2015 (v20150505)
 * The `@expose` annotation is now deprecated. Please use `@export` or `@nocollapse` instead.
 * Removed the "checkStructDictInheritance" warning so that `@struct` classes can extend non-`@struct` classes, and vice versa.
 * Better typechecking for properties assigned in a local scope. See https://github.com/google/closure-compiler/commit/dac0948cd318d9ac572b850964f4451295859422 for details.
 * Several fixes and improvements to the new type inference.
 * Classes defined in a goog.module can now be subclassed.
 * Added the PolymerPass which handles Polymer-specific patterns (thanks to @[jklein24](https://github.com/jklein24))
 * Added a new lint check to make sure invalid types are not used as object keys. See https://github.com/google/closure-compiler/commit/78cd2730d4b201dc63d64c03683dddbb1d098c9e (thanks to @[nbeloglazov](https://github.com/nbeloglazov)).
 * Better warnings for misplaced JSDoc annotations.
 * ES6-related fixes:
    - Transpiled for/of loops now work correctly with native Map and Set implementations.
    - Fixed typedefs in ES6 modules to be renamed correctly.
    - Improved output codesize for ES6 code by ensuring that unused classes are eliminated.
    - Various fixes to --preserve_type_annotations mode (thanks to @[shicks](https://github.com/shicks))


### March 15, 2015 (v20150315)
- Type based optimizations are now enabled by default in advanced mode. They may be disabled with `--use_types_for_optimization false`
- Stricter missing goog.require check
- Fix various crashes
- Bugfixes in transpiling of ES6 modules and classes
- More work on still experimental inline type syntax
- Implement more lint checks in compiler:
  * Extends without goog.require
  * Use of implicitly nullable JSDoc

### February 2, 2015 (v20150126)
- Initial work on supporting TypeScript-style type annotations, with the --language_in=ES6_TYPED flag.
- The type annotation {?T} is now parsed correctly if T is a template type.
- Infer the type of 'this' for functions that are immediately bound.
- Performance improvements in the DisambiguateProperties and AmbiguateProperties passes.
- Removed support for '@type {function(...[Foo])}' (the new syntax is '{function(...Foo)}')
- Added the new ES6 collections (Map, Set, WeakMap, WeakSet) to the standard ES6 externs.
- Removed support for the @notypecheck annotation. Use @suppress {checkTypes} instead.
- Removed the checkProvides DiagnosticGroup. You can now use --jscomp_{error,warning}=missingProvide instead.
- Deprecated the --check_requires flag (use --jscomp_{error,warning}=missingRequire instead).
- Lots of improvements in the new type inference.
- Fixed a bug causing the old type inference to behave differently in Java 8 than it does in Java 7
- Removed some places where the parser was doing unnecessary lookahead, which will make parsing much faster, particularly for cases such as deeply nested array literals.
- Fixed a bug in the transpilation of ES6 for/of loops.

### December 15, 2014 (v20141215)
- General:
  * Improved "smart name removal" runtime by 90% in some cases.
  * Check inheritance inside prototype object literals (github issue #707)
- Closure library support:
  * "missing goog.require" check now validates that implemented interfaces are goog.require'd
  * goog.module exports are now considered constants
- Command line runner:
  * added --rename_prefix_namespace command-line option
- Conformance:
  * framework now allows for naming and extending rules for better composability
  * new NoImplicitlyPublicDecls standard rule
- Parser:
  * warn about misplaced function annotations (@param/@return)
  * Minor parser speed improvements
  * Parse ES6 template literals correctly
- New type inference:
  * function bind and goog.bind inference
  * support for @return on constructors and @this on functions
  * misc fixes

### November 20, 2014 (v20141120)
* Add a warning for mistyping namespaces as Object. (See https://github.com/google/closure-compiler/wiki/A-word-about-the-type-Object for more explanation)
* New JS Conformance check: ban Closure-style top-level declarations from having implicitly public visibility.
* Migrated several lint checks into the compiler
* Default visibility levels in @fileoverview is now supported.
* File level @package no longer applies to goog.provide default namespaces
* Add a warning for misuse of goog.provides/goog.requires
* Fixes for goog.module rewriting
* Fix goog.defineClass when extending a generic class
* function(...[string]) can now be written as function(...string)
* "&&" and "||" are now allowed in @define default value expressions
* New type inference fixes:
  - Made nullable dereference checks suppressible in new type inference
  - Understand primitive unboxing
* A few fixes to ES6 transpilation of generators, spread operators, and synthesized class constructors, as well as updates to the module syntax tracking the spec.

### October 23, 2014 (v20141023)
 - Compiler now allows generics without the dot (e.g. Foo&lt;T&gt;)
 - Treat export as visibility annotation, allowing /** @export {Foo} */
 - Updates to ES6 parser and transpiler to match new ES6 spec
 - Got default externs working with new type inference
 - Moved from json.org to gson
 - More work on @fileoverview visibility annotations (still experimental)
 - Github issues fixed:  #186, #187, #550, #589, #606, #618, #619, #635, #640, #643, #652, #656, #658

### September 23, 2014 (v20140923)
 - Various bug fixes and improvements to the new type inference, including inference of array literals (still off by default).
 - More Type Transformation work.
 - ES6 parser and transpilation fixes, including support for ES6 destructuring.
 - Move common ES6 Math methods to the "implemented" ES6 features in the standard externs
 - More fixes and improvements for goog.module
 - Open-source RefasterJs and the conformance framework.

### August 14, 2014 (v20140814)
 - Fix crash when declaring "var arguments" within a function.
 - RescopeGlobalSymbols generates more compact code.
 - CollapseProperties no longer warns about namespaces redefinitions for "var x = x || {}" or "a.b = a.b || {}"
 - Improved "useless code" warnings
 - Reduce memory consumed by strings in the AST generated by the parser.
 - ES6 parser fixes: semicolons in class definitions, computed properties, etc
 - Progress on ES6 transpilation: generators, default parameters, modules, destructuring, let/const, classes, for/of statements.
 - Performance improvements to the new type inference (still off by default).

### July 30, 2014 (v20140730)
Main changes in this release:
 - Warn if an @param has a '.' in the name.
 - Progress on ES6 transpilation: generators, default parameters, modules, destructuring, let/const, classes, for/of statements.
 - Performance improvements to the new type inference (still off by default).

### June 25, 2014 (v20140625)
Major changes in this release:
 - Variables in externs are copied to the "window" object. This means that properties can be accessed as "window.foo" even if they're only declared in the externs as "var foo;"
 - Added support for goog.module() which helps the transition to ES6 modules.
 - Added support for (non labs-prefixed) goog.defineClass which helps the transition to ES6 classes.
 - Added @package access control mode which means the given variable can be accessed from within the same package (like package-private in Java). Thanks to 20% contributor Brendan Linn.
 - Several fixes to make the compiler work with node.js code, mostly from @nicks.
 - Turn on the InferConst pass, which allows the compiler to do more typechecking and inlining.
 - Flatten module type expressions during module-preprocess.
 - Added --source_map_location_mapping flag. Thanks @terencehonles!
 - Added --output_wrapper_file flag. Thanks @retromodular!
 - Enable GatherExternsFromTypes pass for better handling of typedefs in externs.
 - Removed the old Rhino JS parser, since we are now fully switched over to the new ES6 parser.
 - Lots of work on transpiling ES6 to ES3, as well as running some checks directly on ES6 code. Big thanks to our interns, Matt Loring and Michael Zhou for pushing this work forward.
 - Lots of work on the new type inference system (not enabled by default yet)
 - Fixed Github issues: #110, #431, #432, #435, #439, #477

### May 8, 2014 (v20140508)
 - Improvements to make goog.defineClass easier to use
 - Improvements to the command line option syntax, including file globs and -W/-O flags for warning levels and optimizations, respectively. See [PR #403](https://github.com/google/closure-compiler/pull/403)
 - Added sourceMapInputFiles option, to map from generated JS (for example, from Angular templates) to the original source.
 - New parser is enabled by default.
 - Several improvements to performance and memory usage.
 - Lots of work on the new type inference system (not on by default yet).
 - Initial support for some ES6 features in whitespace-only mode.
 - Fixed issues: [issue 102](https://github.com/google/closure-compiler/issues#issue/102), [issue 119](https://github.com/google/closure-compiler/issues#issue/119), [issue 123](https://github.com/google/closure-compiler/issues#issue/123), [issue 186](https://github.com/google/closure-compiler/issues#issue/186), [issue 310](https://github.com/google/closure-compiler/issues#issue/310), [issue 387](https://github.com/google/closure-compiler/issues#issue/387), [issue 388](https://github.com/google/closure-compiler/issues#issue/388), [issue 389](https://github.com/google/closure-compiler/issues#issue/389), [issue 390](https://github.com/google/closure-compiler/issues#issue/390), [issue 400](https://github.com/google/closure-compiler/issues#issue/400) 

### April 7, 2014 (v20140407)
- Add a warning for the use of goog.base for projects that want to support strict mode in uncompiled code.
- Add "arguments.callee", "arguments.caller", "Function.prototype.arguments" and "Function.prototype.caller" to the "strict" mode checks.
- Have the runtime type checker type-check Object as any object type, possibly with an exotic prototype - not necessarily inheriting from a standard Object.
- Move the checking for 'with' statements into the StrictModeCheck.
- Add an InferConsts pass, and use it demonstrate that it fixes problems with CommonJS aliases (off by default).
- Lots of changes in the new type inference system (not enabled yet in this release)
- A few changes in the new parser (not enabled yet in this release)
- Add a warning for functions with a nullable return type that never return null (off by default)
- Issues fixed: [issue 354](https://github.com/google/closure-compiler/issues#issue/354), [issue 853](https://github.com/google/closure-compiler/issues#issue/853), [issue 1217](https://github.com/google/closure-compiler/issues#issue/1217), [issue 1243](https://github.com/google/closure-compiler/issues#issue/1243), [issue 1244](https://github.com/google/closure-compiler/issues#issue/1244), [issue 1264](https://github.com/google/closure-compiler/issues#issue/1264), [issue 1265](https://github.com/google/closure-compiler/issues#issue/1265), [issue 1260](https://github.com/google/closure-compiler/issues#issue/1260), [issue 1267](https://github.com/google/closure-compiler/issues#issue/1267), [issue 1270](https://github.com/google/closure-compiler/issues#issue/1270)

### March 3, 2014 (v20140303)
- Better inference for polymorphic functions as arguments.
- Improved goog.asserts typing.
- Gather property names from record types in externs (off by default, accessible through Java API as gatherExternsFromTypes option).
- Make cross-module method motion deterministic.
- Remove old code
- In progress: new type inference: generics for functions and classes, type casts.
- In progress: new parser to replace Rhino (currently off by default).
- Fixed [issue 719](https://github.com/google/closure-compiler/issues#issue/719), [issue 926](https://github.com/google/closure-compiler/issues#issue/926), [issue 1032](https://github.com/google/closure-compiler/issues#issue/1032), [issue 1109](https://github.com/google/closure-compiler/issues#issue/1109), [issue 1183](https://github.com/google/closure-compiler/issues#issue/1183), [issue 1198](https://github.com/google/closure-compiler/issues#issue/1198), [issue 1201](https://github.com/google/closure-compiler/issues#issue/1201), [issue 1204](https://github.com/google/closure-compiler/issues#issue/1204), [issue 1205](https://github.com/google/closure-compiler/issues#issue/1205), [issue 1219](https://github.com/google/closure-compiler/issues#issue/1219), [issue 1221](https://github.com/google/closure-compiler/issues#issue/1221), [issue 1222](https://github.com/google/closure-compiler/issues#issue/1222), [issue 1228](https://github.com/google/closure-compiler/issues#issue/1228), [issue 1243](https://github.com/google/closure-compiler/issues#issue/1243).

### January 10, 2014 (v20140110)
- New pass: GatherExternProperties.
- Deleted the RemoveTryCatch pass.
- Includes a work-in-progress new type inference pass.
- Warn about invalid use of id generators.
- Add support for a strict-mode compatible version of goog.base.
- Don't warn about ES3-incompatible property names in externs files.
- Warn about the right class in private-property-access warnings.
- Improvements to the fuzzer.
- Fix [issue 1056](https://github.com/google/closure-compiler/issues#issue/1056), [issue 1131](https://github.com/google/closure-compiler/issues#issue/1131), [issue 1135](https://github.com/google/closure-compiler/issues#issue/1135), [issue 1144](https://github.com/google/closure-compiler/issues#issue/1144), [issue 1147](https://github.com/google/closure-compiler/issues#issue/1147), [issue 1157](https://github.com/google/closure-compiler/issues#issue/1157), [issue 1160](https://github.com/google/closure-compiler/issues#issue/1160), [issue 1166](https://github.com/google/closure-compiler/issues#issue/1166), [issue 1168](https://github.com/google/closure-compiler/issues#issue/1168), [issue 1189](https://github.com/google/closure-compiler/issues#issue/1189).

### November 18, 2013 (v20131118)
- Allow multiple JSDoc tags on the same line.
- Warn for unnecessary casts (disabled by default).
- Add the experimental ES6 parser to our tree; we may make it the default in the future.
- Move compiler to Java 7.
- Initial work on a fuzzer for the compiler.
- Add another smart-name pass before property disambiguation (disabled by default).
- Warn when goog.string.Const.from is not called with a string literal.
- Bug fixes for cross-module method motion.
- Bug fixes for template types.
- Fix [issue 1114](https://github.com/google/closure-compiler/issues#issue/1114), [issue 1116](https://github.com/google/closure-compiler/issues#issue/1116), [issue 1120](https://github.com/google/closure-compiler/issues#issue/1120), [issue 1129](https://github.com/google/closure-compiler/issues#issue/1129).

### October 14, 2013 (v20131014)
- Disable the "rewrite function expressions" pass.
- Improvements in type inference for generics, related to record types and to inheritance.
- Allow unknown and record types in inline type annotations.
- Recognize a function return type as an inline annotation.
- Rotate commutative operators to eliminate parens.
- Fix [issue 1007](https://github.com/google/closure-compiler/issues#issue/1007), [issue 1024](https://github.com/google/closure-compiler/issues#issue/1024), [issue 1047](https://github.com/google/closure-compiler/issues#issue/1047), [issue 1070](https://github.com/google/closure-compiler/issues#issue/1070), [issue 1072](https://github.com/google/closure-compiler/issues#issue/1072), [issue 1073](https://github.com/google/closure-compiler/issues#issue/1073), [issue 1085](https://github.com/google/closure-compiler/issues#issue/1085), [issue 1103](https://github.com/google/closure-compiler/issues#issue/1103).
- Extern updates

### August 23, 2013 (v20130823)
- Experimental export option: exportLocalPropertyDefinitions
- Support of inline type declarations in "var" statements.
- RemoveUnusedClassProperties will now remove properties if a set  exists on a prototype.
- RemoveUnusedClassProperties is now on by default in ADVANCED mode.
- Fix: an 8 year old operator precedence bug (Issue 1062).
- Fix: incorrect variable inlining (Issue 1053)
- CollapseProperties correctness improvements
- Fix: spurious warnings for obviously good logical shifts
- VariableReferenceCheck is now suppressible using existing diagnostic groups.
- Extern updates

### July 22, 2013 (v20130722)
- Stricter missing-property checks for structs.
- Improved type inference for assignments to the prototype property.
- Fix incompatibility between removeUnusedClassProperties and Object.seal, and add the pass to advanced optimizations.
- Open source the code-coverage instrumentation pass.
- Allow constructors with a declared return type to be called without new.
- Bugfixes for goog.inherits used with generic types.
- Better detection of block comments that contain jsdoc tags.
- In ES5 mode, don't quote object literal keys when they're ES3 keywords.
- In ES3 mode, allow ES3 keywords as property names but quote them.
- New jsdoc tag @disposes for use with CheckEventfulDisposal pass to specify particular arguments of a function to be disposed.
- Updated externs for angular, jquery and maps api.
- Fix [issue 1017](https://github.com/google/closure-compiler/issues#issue/1017), [issue 1020](https://github.com/google/closure-compiler/issues#issue/1020), [issue 1021](https://github.com/google/closure-compiler/issues#issue/1021), [issue 1023](https://github.com/google/closure-compiler/issues#issue/1023), [issue 1030](https://github.com/google/closure-compiler/issues#issue/1030), [issue 1033](https://github.com/google/closure-compiler/issues#issue/1033), [issue 1035](https://github.com/google/closure-compiler/issues#issue/1035), [issue 1042](https://github.com/google/closure-compiler/issues#issue/1042), [issue 1043](https://github.com/google/closure-compiler/issues#issue/1043).


### June 3, 2013 (v20130603)
- Produce smaller code for boolean conditional expressions.
- Better type checking for classes that implement or extend generic types.
- More accurate attaching of JSDoc to AST nodes.
- Allow collapse properties to collapse simple global aliases.
- New CheckEventfulObjectDisposal pass for finding memory leaks in JS programs.
- Change RescopeGlobalSymbols to not rewrite variables that do not cross JSModules.
- Fix [issue 965](https://github.com/google/closure-compiler/issues#issue/965), [issue 987](https://github.com/google/closure-compiler/issues#issue/987), [issue 994](https://github.com/google/closure-compiler/issues#issue/994), [issue 1002](https://github.com/google/closure-compiler/issues#issue/1002), [issue 1006](https://github.com/google/closure-compiler/issues#issue/1006), [issue 1008](https://github.com/google/closure-compiler/issues#issue/1008).

### April 11, 2013 (v20130411)
- Improve bad cast detection.
- Add new diagnostic group for @struct/@dict inheritance warnings (checkStructDictInheritance)
- compiler support for goog.define
- experimental optimization: --disambiguate_private_properties
- "strip suffix" improvements
- improved experimental generics: support for user defined classes and interfaces using @template
- removed support for @classTemplate (use @template)
- improved "angular_pass" and bug fixes.
- build type speed improvements for several compiler passes
- new externs: svg, page visibility api, device orientation, device motion events, Intl, Jasmine
- misc extern corrections
- Fixes [issue 921](https://github.com/google/closure-compiler/issues#issue/921), [issue 925](https://github.com/google/closure-compiler/issues#issue/925), [issue 927](https://github.com/google/closure-compiler/issues#issue/927), [issue 931](https://github.com/google/closure-compiler/issues#issue/931), [issue 936](https://github.com/google/closure-compiler/issues#issue/936), [issue 937](https://github.com/google/closure-compiler/issues#issue/937), [issue 942](https://github.com/google/closure-compiler/issues#issue/942), [issue 957](https://github.com/google/closure-compiler/issues#issue/957)

### February 27, 2013 (v20130227)
- Moved to git!
- Added support for inline jsdocs for type annotations
- Don't reorder the condition of an IF if it contains side effects
- Speed up the closurePrimitives pass
- Add new diagnostic group for @struct/@dict inheritence warnings
- Add externs for W3C gamepad
- Support for type declarations with @const
- Function declarations are no longer allowed in 'if' blocks, to be consistent with ecmascript5 strict mode
- Better generics type-checking
- Faster phase-ordering algorithm
- @const on a constructor prevents it from being subclassed when visibility checks are on
- Faster dead-code elimination
- Fixed a bug where type inference wouldn't converge
- Fixes [issue 873](https://github.com/google/closure-compiler/issues#issue/873), [issue 871](https://github.com/google/closure-compiler/issues#issue/871), [issue 864](https://github.com/google/closure-compiler/issues#issue/864), [issue 862](https://github.com/google/closure-compiler/issues#issue/862), [issue 884](https://github.com/google/closure-compiler/issues#issue/884), [issue 874](https://github.com/google/closure-compiler/issues#issue/874), [issue 838](https://github.com/google/closure-compiler/issues#issue/838), [issue 253](https://github.com/google/closure-compiler/issues#issue/253), [issue 889](https://github.com/google/closure-compiler/issues#issue/889), [issue 901](https://github.com/google/closure-compiler/issues#issue/901), [issue 890](https://github.com/google/closure-compiler/issues#issue/890), [issue 915](https://github.com/google/closure-compiler/issues#issue/915), [issue 916](https://github.com/google/closure-compiler/issues#issue/916), [issue 918](https://github.com/google/closure-compiler/issues#issue/918), [issue 919](https://github.com/google/closure-compiler/issues#issue/919), [issue 922](https://github.com/google/closure-compiler/issues#issue/922), [issue 925](https://github.com/google/closure-compiler/issues#issue/925), [issue 927](https://github.com/google/closure-compiler/issues#issue/927), [issue 921](https://github.com/google/closure-compiler/issues#issue/921)

### December 12, 2012 (r2388)
- support for type declarations with @protected and @private
- dead code pruning of cases from switches with constant conditions.
- type checking fixes
- @struct/@dict annotations
- Stable naming improvements
- new warning: misplaced type annotation
- new diagnostic groups: suspiciousCode, cast, misplacedTypeAnnotation
- Added goog.defineClass support
- Added goog.getMsgWithFallback support
- better method templating support
- better IIFE inference
- better unused class removal
- AMD/CommonJS improvements
- Fixes [issue 867](https://github.com/google/closure-compiler/issues#issue/867), [issue 857](https://github.com/google/closure-compiler/issues#issue/857), [issue 851](https://github.com/google/closure-compiler/issues#issue/851), [issue 271](https://github.com/google/closure-compiler/issues#issue/271), [issue 791](https://github.com/google/closure-compiler/issues#issue/791), [issue 841](https://github.com/google/closure-compiler/issues#issue/841), [issue 764](https://github.com/google/closure-compiler/issues#issue/764), [issue 61](https://github.com/google/closure-compiler/issues#issue/61), [issue 808](https://github.com/google/closure-compiler/issues#issue/808), [issue 820](https://github.com/google/closure-compiler/issues#issue/820), [issue 824](https://github.com/google/closure-compiler/issues#issue/824), [issue 804](https://github.com/google/closure-compiler/issues#issue/804), [issue 821](https://github.com/google/closure-compiler/issues#issue/821)

### September 17, 2012 (r2180)
- Turn on es5 strict checks in VERBOSE mode
- Updated library dependencies (like Guava)
- Added a pass for chrome extension i18n
- Bug fixes for String.prototype.split
- warnings whitelist improvements
- heap pressure improvements
- type-checking improvements in inner scopes
- fixes [issue 726](https://github.com/google/closure-compiler/issues#issue/726), [issue 794](https://github.com/google/closure-compiler/issues#issue/794), [issue 783](https://github.com/google/closure-compiler/issues#issue/783), [issue 787](https://github.com/google/closure-compiler/issues#issue/787), [issue 785](https://github.com/google/closure-compiler/issues#issue/785), [issue 730](https://github.com/google/closure-compiler/issues#issue/730), [issue 779](https://github.com/google/closure-compiler/issues#issue/779), [issue 756](https://github.com/google/closure-compiler/issues#issue/756), [issue 777](https://github.com/google/closure-compiler/issues#issue/777), among others

### July 10, 2012 (r2079)
- Better checks for global names (controlled by @suppress {undefinedNames})
- Better checks for dead code
- Many bug fixes around function inlining, variable inlining
- Adds compiler support for inferred type using type templates in function declarations
- Enforce method inheritance even when @override is not specified
- Some native type inference for goog.bind, goog.partial
- Better type inference for inner functions
- Better type inference for goog.asserts functions
- Emit an error for named functions in a goog.scope (the old compiler just emitted corrupt code). Fixes [issue 737](https://github.com/google/closure-compiler/issues#issue/737)
- Fixes to CommonJS modules ([issue 732](https://github.com/google/closure-compiler/issues#issue/732))
- added a command-line flag to the open-source compiler for type-based optimizations
- added type-based property inlining
- Among many bug fixes, fixes [issue 747](https://github.com/google/closure-compiler/issues#issue/747), [issue 748](https://github.com/google/closure-compiler/issues#issue/748), [issue 735](https://github.com/google/closure-compiler/issues#issue/735), [issue 481](https://github.com/google/closure-compiler/issues#issue/481), [issue 728](https://github.com/google/closure-compiler/issues#issue/728), [issue 718](https://github.com/google/closure-compiler/issues#issue/718), [issue 685](https://github.com/google/closure-compiler/issues#issue/685), [issue 714](https://github.com/google/closure-compiler/issues#issue/714), [issue 711](https://github.com/google/closure-compiler/issues#issue/711), [issue 662](https://github.com/google/closure-compiler/issues#issue/662), [issue 773](https://github.com/google/closure-compiler/issues#issue/773), [issue 772](https://github.com/google/closure-compiler/issues#issue/772), [issue 688](https://github.com/google/closure-compiler/issues#issue/688), [issue 768](https://github.com/google/closure-compiler/issues#issue/768), [issue 769](https://github.com/google/closure-compiler/issues#issue/769), [issue 759](https://github.com/google/closure-compiler/issues#issue/759), [issue 753](https://github.com/google/closure-compiler/issues#issue/753), [issue 729](https://github.com/google/closure-compiler/issues#issue/729), [issue 746](https://github.com/google/closure-compiler/issues#issue/746), [issue 724](https://github.com/google/closure-compiler/issues#issue/724)

### April 30, 2012 (r1918)
- experimental support for jquery primitives
- adds the @expose annotation
- Adds --only_closure_dependencies
- sort closure dependencies by default
- improvements to RescopeGlobalSymbols
- improved type inference
- fixed an infinite loop in --ide_mode, and other parser improvements
- fixes for miscellany issues: [issue 684](https://github.com/google/closure-compiler/issues#issue/684), [issue 696](https://github.com/google/closure-compiler/issues#issue/696), [issue 701](https://github.com/google/closure-compiler/issues#issue/701), [issue 700](https://github.com/google/closure-compiler/issues#issue/700), [issue 689](https://github.com/google/closure-compiler/issues#issue/689), [issue 691](https://github.com/google/closure-compiler/issues#issue/691), [issue 672](https://github.com/google/closure-compiler/issues#issue/672), [issue 709](https://github.com/google/closure-compiler/issues#issue/709), [issue 698](https://github.com/google/closure-compiler/issues#issue/698)

### March 5, 2012 (r1810)
- Build-time improvements
- minor peephole rewriting improvements
- Added better warning controls to the Ant task ([issue 640](https://github.com/google/closure-compiler/issues#issue/640))
- Better type inference ([issue 669](https://github.com/google/closure-compiler/issues#issue/669))
- Removal for goog.addSingletonGetter ([issue 668](https://github.com/google/closure-compiler/issues#issue/668))
- fixed a bug in ide_mode ([issue 663](https://github.com/google/closure-compiler/issues#issue/663))
- better method arity checks
- adds support for @suppress {undefinedNames}
- fixes for miscellany issues: [issue 634](https://github.com/google/closure-compiler/issues#issue/634), [issue 651](https://github.com/google/closure-compiler/issues#issue/651), [issue 652](https://github.com/google/closure-compiler/issues#issue/652), [issue 650](https://github.com/google/closure-compiler/issues#issue/650), [issue 656](https://github.com/google/closure-compiler/issues#issue/656), [issue 643](https://github.com/google/closure-compiler/issues#issue/643), [issue 284](https://github.com/google/closure-compiler/issues#issue/284), [issue 582](https://github.com/google/closure-compiler/issues#issue/582), [issue 647](https://github.com/google/closure-compiler/issues#issue/647), [issue 310](https://github.com/google/closure-compiler/issues#issue/310), [issue 368](https://github.com/google/closure-compiler/issues#issue/368)

### January 23, 2012 (r1741)
- Compiler now preserves code with hidden side-effects in some cases. Addresses [issue 64](https://github.com/google/closure-compiler/issues#issue/64), [issue 398](https://github.com/google/closure-compiler/issues#issue/398)
- In Advanced mode, the compiler is now more aggressive about removing unused properties (RemoveUsedClassProperties).
- Improved jQuery 1.7, Google Maps 3.7, Typed Array and other externs definitions
- Experimental AMD and Common JS module support.
- Parser improvements.
- Better unused variable removal. [issue 641](https://github.com/google/closure-compiler/issues#issue/641)
- Better support for @lends. [issue 314](https://github.com/google/closure-compiler/issues#issue/314)
- Better type checking around conditionals, "Function.bind"/ "goog.bind", etc
- Source maps now use the v3 format by default.
- Removed some obsolete options from CompilerOptions
- Fixes [issue 644](https://github.com/google/closure-compiler/issues#issue/644), [issue 641](https://github.com/google/closure-compiler/issues#issue/641), [issue 638](https://github.com/google/closure-compiler/issues#issue/638), [issue 621](https://github.com/google/closure-compiler/issues#issue/621) [issue 619](https://github.com/google/closure-compiler/issues#issue/619), [issue 618](https://github.com/google/closure-compiler/issues#issue/618), [issue 603](https://github.com/google/closure-compiler/issues#issue/603), [issue 601](https://github.com/google/closure-compiler/issues#issue/601), [issue 600](https://github.com/google/closure-compiler/issues#issue/600), [issue 583](https://github.com/google/closure-compiler/issues#issue/583), [issue 575](https://github.com/google/closure-compiler/issues#issue/575), [issue 314](https://github.com/google/closure-compiler/issues#issue/314) among others

### November 14, 2011 (r1592)
- Fixed support -0.0, [issue 582](https://github.com/google/closure-compiler/issues#issue/582).
- Added support for compile time defines in ant builds, [issue 567](https://github.com/google/closure-compiler/issues#issue/567)
- Fixes for [issue 588](https://github.com/google/closure-compiler/issues#issue/588), [issue 586](https://github.com/google/closure-compiler/issues#issue/586), [issue 581](https://github.com/google/closure-compiler/issues#issue/581), [issue 569](https://github.com/google/closure-compiler/issues#issue/569), [issue 567](https://github.com/google/closure-compiler/issues#issue/567), [issue 566](https://github.com/google/closure-compiler/issues#issue/566), [issue 558](https://github.com/google/closure-compiler/issues#issue/558), [issue 550](https://github.com/google/closure-compiler/issues#issue/550), [issue 544](https://github.com/google/closure-compiler/issues#issue/544), [issue 539](https://github.com/google/closure-compiler/issues#issue/539), [issue 531](https://github.com/google/closure-compiler/issues#issue/531),
- New jQuery 1.7 and Gadget API externs
- Internal AST cleanup, removed unused Token and annotation IDs, unused legacy Rhino classes.
- Added experimental [RescopeGlobalSymbols](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/RescopeGlobalSymbols.java) pass.
- Add `checkDebuggerStatement` diagnostic group to enable checks for `debugger` statements
- Minor type checking improvements
- Minor optimization improvements
- Updated Guava and Ant dependencies to the latest available
- Webservice support for EcmaScript 5 and 5 Strict


### October 3, 2011 (r1459)
- Bad function inlining bug fixes
- Type system handles more idioms for assigning to the prototype ([issue 537](https://github.com/google/closure-compiler/issues#issue/537))
- Fixes compiler crash in UnreachableCodeElimination and InlineObjectLiterals ([issue 545](https://github.com/google/closure-compiler/issues#issue/545))
- Easier to suppress visiblity warnings
- Extend ExportTestFunctions so that it properly exports test methods defined on objects.
- Various type-checking bug fixes
- The module flags now lets you specify a %basename% placeholder with the name of the module output file.
- Adds an experimental option for more aggressive function inlining
- Other assorted issue fixes: 548, 533. 457, 538, 535, 534, 446, 511, 530, 529, 528

### August 11, 2011 (r1346)
- Bug fixes for using stdin as input that was introduced last release
- Bug fix for live variable analysis to handle expression in for-in loops
- Improve output delimiter to escaped when needed.
- Better warnings for property disambiguation

### August 4, 2011 (r1314)
- Checks for ecmascript5/strict mode
- Improvements to type checking
- Fixed some over-aggressive optimizations in advanced mode
- Local collapsing of object literals
- Extern updates
- [Fix issues 521, 505, 504, 501, 490, 489, 488 and others](http://code.google.com/p/closure-compiler/issues/list?can=1&q=closed-after%3A2011%2F6%2F15+closed-before%3A2011%2F8%2F2+status%3AFixed&colspec=ID+Type+Status+Priority+Component+Owner+Summary&cells=tiles)

### June 15, 2011 (r1180)
- Improvements to line-number reporting
- Interfaces can extend multiple interfaces, with checks for conflicting signatures
- Support for goog.typedef has been removed. Use the @typedef annotation.
- Closure namespaces are @const by default
- Optimization of parseInt, parseFloat
- Emit a warning if a private property overrides another private property
- Make sure functions are called with an appropriate 'this' type
- Add warnings about parameters being reassigned in a way that violates their type definition
- Add warnings if a object has properties defined before the object is defined
- Add duplicate object literal key check to jscompiler es5 strict mode checks
- Update jQuery 1.6 externs
- Fixes issues 459, 477, 482, 486, and many others

### May 2, 2011 (r1043)
- Now emits a warning if a block comment looks like it should be a jsdoc comment.
- Type checking improvements around interfaces.
- Improved handling of constructors defined in object literals.
- 20% smaller source maps (version 2).
- Experimental source map improvements (version 3)
- Rewrite goog.object.create at compile time
- Fix [issue 380](https://github.com/google/closure-compiler/issues#issue/380),407,412,413,416,423, 428

### April 5, 2011 (r964)
- Workaround Firefox parser, [issue 397](http://code.google.com/p/closure-compiler/issues/detail?id=397)
- Workaround Opera parser issue with compiled jQuery, [issue 390](http://code.google.com/p/closure-compiler/issues/detail?id=397)
- Workaround IE handling of vertical tabs in string to number conversions.
- Correct build time performance, [issue 386](http://code.google.com/p/closure-compiler/issues/detail?id=386)
- Fix crash compiling Dojo, [issue 389](http://code.google.com/p/closure-compiler/issues/detail?id=389)
- Improved folding of array and object literals
- [Fixed issues 49, 171, 304, 329, 345, 378, 379, 389, 390, 392, 397, 399](http://code.google.com/p/closure-compiler/issues/list?can=1&q=closed-after%3A2011%2F3%2F22+closed-before%3A2011%2F4%2F5+status%3AFixed)

### March 22, 2011 (r916)
- Added support for @interface declarations with non-function members
- Added support for check @const property definitions
- Deprecation warnings are now on by default with VERBOSE warnings
- Missing return warnings are now on by default with VERBOSE warnings
- Improved variable renaming algorithm
- Preliminary ECMASCRIPT5/ECMASCRIPT5_STRICT support (--language_in command line flag)
- Compile-time performance issue (Issue 349)
- More constants folding improvements
- Optimizations on the call graph are now performed in the main optimization loop
- goog.asserts are stripped by default, they can be enabled with the --debug option
- Improved unused code removal
- Other flag changes: --output_wrapper_marker support was removed, --flagfile support added
- New extern: jQuery 1.5, Google Maps API 3.4, Facebook JavaScript SDK, WebGL, IndexDB
- [Fixed issues 387, 384, 383, 381, 378, 375, 372, 369, 367, 366, 364, 363, 353, 349, 348, 325, 322, 321, 319, 318, 315, 301, 269, 266, 251, 249, 247, 204, 195, 133](http://code.google.com/p/closure-compiler/issues/list?can=1&q=closed-after%3A2011%2F1%2F19+closed-before%3A2011%2F3%2F22+status%3AFixed+)

### January 19, 2011
- whitespace-only support for es5 getters and setters
- New type syntax for structural constructors "{function(new:Type)}"
- Support for the @lends annotation.
- Add a @suppress {uselessCode} warnings group
- Improvements to constant-folding
- Native handling for goog.tweaks
- Emits an error if the left-hand side of an assign isn't an assignable expression
- removal of redundant returns and throws
- Optimize the "arguments" variable when possible.
- "toString" and "valueOf" are now officially assumed to be side-effect free.
- Add a --closure_entry_point flag, for use in conjunction with --manage_closure_dependencies
- Assorted type-checking fixes
- Fixes issues 311, 268, 267, 248, 261, 258, 187, 255, 236, 244, among others

### September 17, 2010
- Full support for goog.scope
- Better optimization of external calls that have no side effects (like getElementById)
- Much better unused var removal
- A number of peephole and constant-folding optimizations
- Much faster source map generation
- Better property detection for missingProperties check
- Much better type inference in local scopes
- type inference of return types on functions
- type checker warns about comparison of functions  to booleans, numbers, or strings
- Better externs file validation
- @suppress annotations work for all canonical warning groups
- @notypecheck annotation blocks more warnings
- General performance improvements
- Fixes issues 188, 186, 96, 177, 143, 221, 205, 200, 197, 182, 194, 172, 191, 71, 66, 229, among others

### June 16, 2010
- Beta support for goog.scope
- Smarter unreachable code analysis.
- Folds "new Error" to "Error", "new RegExp" to "RegExp", etc.
- Extract Prototype Member / Devirtualization improvements
- deterministic source maps
- Peephole optimizations
- Added @nocompile annotation to work with --manage_closure_dependencies
- Inline more functions into the global scope in advanced mode
- Better detection for dangerous 'this' references in verbose mode
- Disabled flow sensitive variable inlining by default, because it still has problems
- Fixes issues 174, 166, 168, 125 among others


### May 14, 2010
- Shadowing of the `arguments` variable is now forbidden
- Added the `--manage_closure_dependencies` flag (pending documentation)
- Support for the @extern annotation
- Type-inference plugin that knows about goog.asserts
- Data-flow based variable inlining
- Better function inlining
- Warnings for unknown @suppress parameters
- Fixes for [issue 141](https://github.com/google/closure-compiler/issues#issue/141), [issue 139](https://github.com/google/closure-compiler/issues#issue/139), [issue 58](https://github.com/google/closure-compiler/issues#issue/58), [issue 112](https://github.com/google/closure-compiler/issues#issue/112), and many others

### Mar 30, 2010
- Renamed `CompilerRunner.java` to `CommandLineRunner.java`
- More user-friendly messaging for trailing commas
- Improvements to inlining and property ambiguation optimizations
- Run-time type-checking (not available from the command-line)
- Better type checking in inner functions
- Fixes for [issue 33](https://github.com/google/closure-compiler/issues#issue/33), [issue 103](https://github.com/google/closure-compiler/issues#issue/103), [issue 116](https://github.com/google/closure-compiler/issues#issue/116), [issue 124](https://github.com/google/closure-compiler/issues#issue/124), [issue 127](https://github.com/google/closure-compiler/issues#issue/127), and [issue 130](https://github.com/google/closure-compiler/issues#issue/130) (among others)

### Feb 1, 2010
- Support for Ant (BuildingWithAnt)
- The ability to turn type warnings on/off independent of other flags (--jscomp_warning=checkTypes)
- With --warning_level=QUIET, all warnings will be silenced.
- Fix for Issue 86 ("@inheritDoc doesn't play well with interfaces")
- Fix for Issue 81 ("eval function replaced with eval operator")
- Fix for Issue 80 ("Inappropriate side-effect warning")
- A few improvements to variable inlining.
 

### Dec 17, 2009:
- Fix for [issue 75](https://github.com/google/closure-compiler/issues#issue/75) ("indexing into an array with a string shouldn't be an error")
- Fix for [issue 63](https://github.com/google/closure-compiler/issues#issue/63) ("create source map in whitespace-only mode.")
- Fix for [issue 24](https://github.com/google/closure-compiler/issues#issue/24) ("Add --charset option to command line compiler that changes the input and output chararacter set.")
- Don't generate warnings for ES5 directives
- Generate a warning for unsafe uses of "with".  Suppress warning with  /** @suppress {with} */
- Various bugs fixed.
  
### Dec 3, 2009:

- Several bug fixes.
- Make --define part of the open source API.
- Allow RemoveUnusedVars to strip unused anonymous function names.
- Add a warning for function declarations that may behave differently in different browsers.
- Enable local variable inlining for simple mode.
- Digits aren't allowed as the first character of a key, so add a prefix.

### November 19, 2009:

- Better dead assignment elimination
- Support for goog.base
- Allow $ in method names.
- Inline local functions and anonymous functions
- Fix some compiler crashes when folding ifs and handling local vars
- Better type inference in externs
- Don't emit warnings for JSDoc annotations we don't support.