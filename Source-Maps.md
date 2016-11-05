# Source Map documentation

Closure Compiler can produce source maps to aid in debugging. These are created when the `--create_source_map` flag is used. While the compiler also has the `--souce_map_input` flag which allows the compilation to be aware of previous transformations, input source maps are only utilized to display error messages and do not affect the output source map.

You can pass source maps for input files to the compiler with the `--source_map_input` flag. The npm gulp plugins utilize the gulp-sourcemaps plugin.

### Notes regarding source map generation and consumption.

Design docs here:

[Source Map v3](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?hl=en_US)

Design docs for deprecated versions:

[Source Map v2](https://docs.google.com/document/d/1xi12LrcqjqIHTtZzrzZKmQ3lbTv9mKrN076UB-j3UZQ/edit?hl=en_US)

[Source Map v1](https://docs.google.com/a/google.com/document/d/1g6tuP7unEkxUSZwLm4IcLoJn1eNDhEmZLAV2kphdvOY/edit)

Implementation here:

[com.google.debugging.sourcemap](https://github.com/google/closure-compiler/blob/master/src/com/google/debugging/sourcemap)

Mozilla's JS Implementation here:

https://github.com/mozilla/source-map

Webkit JS Implementation here:

http://trac.webkit.org/browser/trunk/Source/WebInspectorUI/UserInterface/Models/SourceMap.js#L158

Blink JS Implementation here:

https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/devtools/front_end/sdk/SourceMap.js&type=cs&l=282

Discussions:

dev-js-sourcemap@lists.mozilla.org 

Chrome's support of Source Maps discussed:

http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/


A SourceMap encoding/decoding debugger:
http://murzwin.com/base64vlq.html
