# JavaScript supported by Closure Compiler

By default, Closure compiler supports JavaScript as specified in [Ecma-262, edition 10 (ECMAScript 2019)](https://www.ecma-international.org/ecma-262/10.0/index.html).

Support for ECMAScript 2020 features is in progress.

Here are some pages that we find useful for understanding the language:
- [Mozilla's Core JavaScript Reference](http://developer.mozilla.org/en/docs/Core_JavaScript_1.5_Reference)
- [Differences between IE's JScript implementation and JavaScript (PDF)](http://wiki.ecmascript.org/lib/exe/fetch.php?id=resources%3Aresources&cache=cache&media=resources:jscriptdeviationsfromes3.pdf)

Using the command-line options `--language_in`, the compiler can be switched to accept any version of ECMAScript down to [Ecma-262, edition 3 (ECMAScript 3) \(PDF\)](https://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262,%203rd%20edition,%20December%201999.pdf). See the [flags page](https://www.ecma-international.org/ecma-262/10.0/index.html) for a full list of supported `--language_in` values.