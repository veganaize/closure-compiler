## Migration from `goog.module` to ES6 modules

### Closure module default exports

Closure modules can have a "default" export via `export = value;`. When using
this syntax the module's exports become the default export. This is frequently
used with modules that export a single class.

```javascript
goog.module('my.Class');

class Class {}

// Default export. goog.require for this module returns this class.
exports = Class;
```

```javascript
goog.module('my.namespace');

// Non-default exports. goog.require for this module returns an object that
// has keys CONSTANT_0 and CONSTANT_1.

/** @const {number} */
exports.CONSTANT_0 = 0;
/** @const {number} */
exports.CONSTANT_1 = 1;
```

It should be noted that this is **not** the same as an ES6 module's default
export, especially when it comes to `goog.require`. When `goog.require`ing an
ES6 module it **always** returns the module object (like `import * as`). If the
ES6 module has a default export then that means the object will have a key
`default` with the default export as value. Closure module default exports
directly change what `goog.require` returns. ES6 default exports always result in
`goog.require` returning an object with both any named exports and the default
export.

```javascript
// This Closure module actually emulates an ES6 module's default export because
// it has an export named "default".
goog.module('my.namespace');

/** @const {number} */
exports.default = 0;
```

As a result it is harder to migrate Closure modules with default exports than
those without. Note that this is just the general rule and _some_ default
exports function like ES6 modules and thus are still easy to convert. If the
default export is itself _module-like_ (a static object with static keys), then
it is equivalent to not having default exports and should also be easy to
convert. For example, these two modules are the same even though one uses
default exports and the other does not.

```javascript
goog.module('my.namespace');

const THE_ANSWER = 42;

exports = {THE_ANSWER};
```

```javascript
goog.module('my.namespace');

/** @const */
exports.THE_ANSWER = 42;
```

### Migrating Closure modules with named exports

For Closure modules that use named exports we recommend migrating files in place
over 3 changes.

1.  Edit the implementation file to be an ES6 module that calls
    `goog.declareModuleId` with the old Closure module name (i.e. the old
    argument to `goog.module`). This should allow other files to continue using
    `goog.require` on the Closure namespace.
    -   Be sure to import Closure's `goog.js` file rather than use the global
        `goog` symbol!
1.  Edit all ES6 module files that are using `goog.require` on the module
    namespace to instead use ES6 `import`.
1.  Remove the call to `goog.declareModuleId` in the migrated file once it is
    not being referenced by any `goog.module` or `goog.provide` files.

### Migrating Closure modules with default exports {#closure-module-with-default-exports}

For Closure modules that do have default exports we recommend migrating files
over 4 changes. This migration cannot be done purely in place. It requires adding
new files.

1.  Edit the implementation file to be an ES6 module that calls
    `goog.declareModuleId` with a new namspace. Create a new file \*\_shim.js
    next to it. This new file should be a Closure module with the original
    Closure namespace and should `goog.require` the ES6 module and forward its
    exports. Remember that your `goog.require` returns an ES6 module object and
    you want to forward just one of its keys with `export = value.`
1.  Edit all ES6 module files that are using `goog.require` on the module
    namespace to instead use ES6 `import`.
1.  Delete the \*\_shim.js file and change any left over `goog.require`s in
    Closure modules to the new namespace.
1.  Remove the call to `goog.declareModuleId` in the migrated file once it is
    not being referenced by any `goog.module` or `goog.provide` files.

## Other

### What about migrating Closure modules that call `declareLegacyNamespace`?

`goog.module.declareLegacyNamespace` is not supported in ES6 modules. It breaks
a fundamental principle of modules: that modules do not create global values.
Its purpose was to allow migration from `goog.provide` to Closure modules. We do
not support it when moving to ES6 modules.

It is recommended you migrate all `goog.provide`d files that are relying on the
`declareLegacyNamespace` call to Closure or ES6 modules first, or at the very
least have them call `goog.module.get` in a `goog.scope` to get references to
modules. If this is not possible and migration of the Closure module is still
desired, you'll need to follow similar steps to
[migrating a Closure module with default exports](#closure-module-with-default-exports),
except your \*\_shim.js file will call `declareLegacyNamespace`. You will only
be able to delete this file once all the `goog.provide` files have migrated.
