How to create HTML documentation from a closure-compiler project by using the JSDoc tool:

The JSDoc project at http://usejsdoc.org is a kind of distant cousin to the closure-compiler project.   Some recent (January 2015) changes to JSDoc have made it possible to use JSDoc on closure-compiler based projects.  Specifically, the new JSDoc 3.3.0 version has better support for the `@interface` tag and inheritance among interfaces and classes, so that the `@inheritDoc` tag is correctly recognized.

Note that there are a couple of other projects aimed at generating docs for closure-compiler projects.  See  [js-dossier](https://github.com/jleyba/js-dossier) and [a discussion about closure docs](https://groups.google.com/forum/#!topic/closure-library-discuss/ElI-lAiUqFA).

## Changes to Adapt to JSDoc

Here is a summary of changes needed to adapt a closure-compiler based project to work with JSDoc 3.3.0.

* Add `@classdesc` to constructor of classes and interfaces.  This is where to put the summary docs for the class or interface.  Anything coming before the `@classdesc` will appear as the docs for the constructor itself.

* Add `@method` to each method definition of an `@interface`.  Not needed for methods defined in a regular (non-interface) class.

* Add `@implements` as needed for classes/interfaces that inherit from another class/interface (JSDoc doesn't always infer this).

* Add `@static` to static functions or properties.

* To deal with `goog.scope`, use `@alias` to tell JSDoc what's happening.  The `@ignore` tells JSDoc to not produce separate docs for the short-name variable.  Here is the pattern:

```javascript
    goog.scope(function() {

        /**
        * @constructor
        */
        myproject.packageA.Foo = function() {
        };

        /**
        * @alias myproject.packageA.Foo
        * @ignore
        */
        var Foo = myproject.packageA.Foo;

        /**
        * @return {number}
        */
        Foo.prototype.makeBar = function() {
          return 0;
        };

    }); // goog.scope
```

* When compiling with closure compiler, add this compiler option:
```
    --extra_annotation_name=alias
```

* All types in `@param` and `@return` statements should be full path names. Don't use a `goog.scope` short-name there.


* Specify "closure" in the `tags.dictionaries` config property.  See http://usejsdoc.org/about-configuring-jsdoc.html.  Here is a sample `jsdoc_configure.json` file showing this

```json
    {
        "tags": {
            "allowUnknownTags": true,
            "dictionaries": ["jsdoc","closure"]
        },
        "source": {
            "include": [
                "src/"
            ],
            "includePattern": ".+\\.js(doc)?$"
        },
        "plugins": ["plugins/markdown"],
        "templates": {
            "cleverLinks": true,
            "monospaceLinks": false,
            "default": {
                "outputSourceFiles": true,
                "layoutFile": "src/docs/layout.tmpl"
            }
        }
    }
```

* You can modify the CSS used with the default template by specifying a  `templates.default.layoutFile` config property to be a custom `layout.tmpl` document.  See https://github.com/jsdoc3/jsdoc/issues/480 and https://github.com/jsdoc3/jsdoc3.github.com/issues/47.  The sample `jsdoc_configure.json` file shown above uses this option.


## Current Problems Using JSDoc

As of April 2015 below are listed some problems of using JSDoc with closure-compiler.    See the [JSDoc issues list](https://github.com/jsdoc3/jsdoc/issues) for more information.

* [Closure Compiler Punch List](https://github.com/jsdoc3/jsdoc/issues/605) tracks all of the Google Closure Compiler annotations that JSDoc does not currently recognize. 

* Some cases where interfaces inherit from other interfaces can result in documentation not appearing where it should.  See https://github.com/jsdoc3/jsdoc/issues/912

* You cannot mark a constructor with `@private`.  See https://github.com/jsdoc3/jsdoc/issues/958 and https://github.com/jsdoc3/jsdoc/issues/952

* `@package` access is not recognized.  See https://github.com/jsdoc3/jsdoc/issues/962

* The link to source code in the docs points to the `@interface` instead of the actual code.  See https://github.com/jsdoc3/jsdoc/issues/895

* The "template" part of JSDoc is not easy to modify and has very little documentation.  See https://github.com/jsdoc3/jsdoc.  There is a project underway called [JSDoc-Baseline](https://github.com/hegemonic/jsdoc-baseline) which is meant to be an extensible template for JSDoc3.