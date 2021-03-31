NOTE: This page started life as a copy of a [stackoverflow article](https://stackoverflow.com/questions/10395810/how-do-i-split-my-javascript-into-modules-using-googles-closure-compiler) by ChadKillingsworth.

# Overview of dynamic loading and chunks

By default the output from closure-compiler is one long JS script that combines all of the input source code after optimizations have been applied. However, this may not work for you if your application is really big. In that case, you may want to have the compiler break your code up into multiple chunks that can be loaded separately. You will design your application so that the code you always need gets loaded in an initial chunk, probably from a `<script>` tag, then that chunk will load others (which may load still others) as needed in order to support the user's actions. (e.g. The user may request a new display view, which requires you to load the code for showing that view.)

# Asking closure-compiler to produce chunks

There are multiple compiler flags pertaining to chunks.

Each use of the --chunk flag describes an output file and it's dependencies. Each chunk flag follows the following syntax:

    --js inputfile.js
    --chunk name:num_files:dependency

The resulting output file will be name.js and includes the files specified by the preceding --js flag(s).

The dependency option is what you will be most interested in. It specifies what the parent chunk is. The chunk options must describe a valid dependency tree (you must have a base chunk).

Here's an example:

    --js commonfunctions.js
    --chunk common:1
    --js page1functions.js
    --js page1events.js
    --chunk page1:2:common
    --js page2function.js
    --chunk page2:1:common
    --js page1addons.js
    --chunk page1addons:1:page1

In this case, you are telling the compiler that the page1 and page2 chunks depend on the common chunk and that the page1addons chunk depends on the page1 chunk.

Keep in mind that the compiler can and does move code from one chunk into other chunk output files if it determines that it is only used by that chunk.

None of this requires closure-library or the use of goog.require/provide calls nor does it add any code to your output. If you want the compiler to determine dependencies automatically or to be able to manage those dependencies for you, you'll need to use a module format such as CommonJS, ES2015 modules or goog.require/provide/module calls.

See also https://github.com/chadkillingsworth/closure-calculate-chunks for a tool that will help you automate the process of deciding which source files should end up in which chunks.

# Loading the chunks at runtime

The chunks are intended to be loaded as scripts, not JS modules.
If you load them as JS modules, you may have problems with some global variables not being available when you expect them to be.
(See e.g. https://github.com/google/closure-compiler/issues/3752).

Although you can use whatever mechanism you like to trigger loading chunks.
Closure library provides a `goog.module.ModuleManager` class and related classes for this purpose.

NOTE: For a long time we used the term "module" to refer to these separately-loadable chunks, but that was too easily confused with `goog.module()` and ES modules which came along later. The Closure library namespace and class names reflect this older naming.