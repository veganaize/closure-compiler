NOTE: This page started life as a copy of a [stackoverflow article](https://stackoverflow.com/questions/10395810/how-do-i-split-my-javascript-into-modules-using-googles-closure-compiler) by ChadKillingsworth.

See also https://github.com/chadkillingsworth/closure-calculate-chunks for a tool that will help you automate the process of deciding which source files should end up in which chunks.

Closure-compiler's ability to create multiple output files provides a powerful tool to separate input files into distinct output chunks. It is designed such that different chunks can be loaded at differing times depending on the features required. There are multiple compiler flags pertaining to chunks.

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

**Update Note:** Prior to the 20180610 version, the `chunk` flags were named `module`. They were renamed to reduce confusion with proper JS modules. The answer has been updated to reflect the new names.