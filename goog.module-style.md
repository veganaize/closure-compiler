# goog.module export styles

In a `goog.module` file, there are two distinct styles, based on the two styles
of exports in [ES6 modules](http://exploringjs.com/es6/ch_modules.html). Note
that unlike with ES6 modules, goog.module cannot use both styles in a single
module; each goog.module file is considered to be using either a default export
or named exports.


## Default exports

In goog.module, a default export can be created by assigning directly to the
export of a module, and is the most common export style:
```javascript
goog.module('a.b.c.Foo');

class Foo {}

exports = Foo;
```

## Named exports

Named exports from a goog.module can be defined in one of two ways, the first way is
to make the exported name clear at the declaration site, like the following:
```javascript
goog.module('d.e.f');

exports.value = 5;

class Foo {}
exports.Foo = Foo;

```

The other way to list all the exports at the end of the module, with a style
similar to the _revealing module pattern_, like the following:
```javascript
goog.module('d.e.f');

const value = 5;

class Foo {}

exports = {Foo, value};
```

Note that only object literals that contain only names can be used for named
exports in this way, as that is the restriction for the similar syntax
in ES6 module exports.

#### Destructuring imports

One advantage to using named exports is that they can be imported using
destructuring imports to bring in the names of interest as unqualified names
in the importing module, such as:
```javascript
goog.module('x.y.z');

const {value} = goog.require('d.e.f');

alert(value);
```