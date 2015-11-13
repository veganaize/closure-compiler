# Frequently asked questions.

## Table of Contents
### Unexpected Output
 * [The compiler crashes with UnsupportedClassVersionError or Unsupported major.minor version 51.0](#the-compiler-crashes-with-unsupportedclassversionerror-or-unsupported-majorminor-version-510)
 * [I get a "Trailing comma is not legal" error. But it works on Firefox!](#i-get-a-trailing-comma-is-not-legal-error-but-it-works-on-firefox)
 * [I get "invalid property id" errors. But it works on Firefox!](#i-get-invalid-property-id-errors-but-it-works-on-firefox)
 * [I get "Unsupported syntax" errors. But it works on Firefox!](#i-get-unsupported-syntax-errors-but-it-works-on-firefox)
 * [Closure Compiler inlined all my strings, which made my code size bigger. Why did it do that?](#closure-compiler-inlined-all-my-strings-which-made-my-code-size-bigger-why-did-it-do-that)
 * [JSC_VAR_MULTIPLY_DECLARED_ERROR occur using the standalone compiler jar but not with the webservice](#jsc_var_multiply_declared_error-occur-using-the-standalone-compiler-jar-but-not-with-the-webservice)
 * [My code broke when using advanced optimizations! How do I figure out what's wrong?](#my-code-broke-when-using-advanced-optimizations-how-do-i-figure-out-whats-wrong)
 * [Some of my properties are getting renamed, but some aren't. Why?](#some-of-my-properties-are-getting-renamed-but-some-arent-why)
 * [When using type-checking, sometimes Closure Compiler doesn't warn me about missing properties. Why not?](#when-using-type-checking-sometimes-closure-compiler-doesnt-warn-me-about-missing-properties-why-not)

### Using Closure Compiler
 * [What are the recommended Java VM command-line options?](#what-are-the-recommended-java-vm-command-line-options)
 * [How do I use Advanced Optimizations?](#how-do-i-use-advanced-optimizations)
 * [I want to use Advanced Optimizations, but { jQuery, YUI, Underscore, Prototype, JS Library Foo } does not work with them. What do I do?](#i-want-to-use-advanced-optimizations-but--jquery-yui-underscore-prototype-js-library-foo--does-not-work-with-them-what-do-i-do)
 * [Do I need to write type annotations to take advantage of Advanced Optimizations?](#do-i-need-to-write-type-annotations-to-take-advantage-of-advanced-optimizations)
 * [When using Advanced Optimizations, Closure Compiler adds new variables to the global scope. How do I make sure my variables don't collide with other scripts on the page?](#when-using-advanced-optimizations-closure-compiler-adds-new-variables-to-the-global-scope-how-do-i-make-sure-my-variables-dont-collide-with-other-scripts-on-the-page)
 * [How do I call Closure Compiler from the Java API?](#how-do-i-call-closure-compiler-from-the-java-api)
 * [How do I write an externs file?](#how-do-i-write-an-externs-file)
 * [How do I add closure-compiler functionality to my project?](#how-do-i-add-closure-compiler-functionality-to-my-project)
 * [What version of Closure Compiler should I use?](#what-version-of-closure-compiler-should-i-use)
 * [Source Maps and sourceMappingURL](#source-maps-and-sourcemappingurl)

### Contributions
 * [How do I submit a patch?](#how-do-i-submit-a-patch)
 * [How do I submit a bug report?](#how-do-i-submit-a-bug-report)
 * [How do I submit a feature request for a new type of optimization?](#how-do-i-submit-a-feature-request-for-a-new-type-of-optimization)

### Design of Closure Compiler
 * [How do I find out more about how Closure Compiler works?](#how-do-i-find-out-more-about-how-closure-compiler-works)
 * [Is there a specification for the JSDoc type language?](#is-there-a-specification-for-the-jsdoc-type-language)
 * [Can you add alternative syntax X to your JSDoc type language?](#can-you-add-alternative-syntax-x-to-your-jsdoc-type-language)
 * [I'm writing a Firefox extension. It only needs to work on the bleeding-edge version of Firefox. I don't care if it doesn't work in other browsers. Could we add a --platform=FIREFOX flag that lets me use the snazzy features introduced in ?](#im-writing-a-firefox-extension-it-only-needs-to-work-on-the-bleeding-edge-version-of-firefox-i-dont-care-if-it-doesnt-work-in-other-browsers-could-we-add-a---platformfirefox-flag-that-lets-me-use-the-snazzy-javascript-features-introduced-in--javascript-17)
 * [My question isn't covered on this list!](#my-question-isnt-covered-on-this-list)

## Unexpected Output

### The compiler crashes with `UnsupportedClassVersionError` or `Unsupported major.minor version 51.0`

Closure Compiler requires [Java 7](http://www.oracle.com/technetwork/java/javase/downloads/index.html). This cryptic error message is Java's way of telling you to upgrade your version of Java.

### I get a "Trailing comma is not legal" error. But it works on Firefox!

EcmaScript 3 (the JavaScript standard prior to Dec 2009) did not specify the behavior, so browser implementations varied. In Internet Explorer versions prior to IE 9, the object literal `{key: value,}` is an error. 

Different browsers may parse array literals with trailing commas successfully, but differently.

    [1,].length
    => 1 (on Firefox)
    => 2 (on IE)

This difference is highly likely to lead to bugs, so by default we just forbid it outright.  

EcmaScript 5 standardized the behavior to allow trailing commas, and the latest releases of the major browser vendors support it.  The Closure Compiler can accept and generate EcmaScript 5 code (with `--language_in=ES5` or `--language_in=ES5_STRICT`), however as significant number of older browsers are still in use, it is not the default.

### I get "invalid property id" errors. But it works on Firefox!

As an example, this code:

     ({float:"left"});

will cause a parse error like:

      JSC_INVALID_ES3_PROP_NAME: Keywords and reserved words are not allowed as unquoted property names in older versions of JavaScript. If you are targeting newer versions of JavaScript, set the appropriate language_in option. at line 1 character 2
      ({float:"left"});
        ^

EcmaScript 3 (the JavaScript standard prior to Dec 2009) did not allow keywords or reserved keywords to be used as unquoted property names.  EcmaScript 5 changed this and now allows them and the latest releases of the major browser vendors support it.  The Closure Compiler can accept and generate EcmaScript 5 code, however as significant number of older browsers are still in use, it is not the default.

If you are only concerned with WHITESPACE or SIMPLE optimization modes, you can fix this without changing the language mode by quoting the property:

     ({"float":"left"});

In ADVANCED mode you need to be sure to quote the property references consistently, be sure to read the [https://developers.google.com/closure/compiler/docs/api-tutorial3 documentation](1,]`). 

### I get "Unsupported syntax" errors. But it works on Firefox!

Firefox has done a lot of cool innovations since JavaScript 1.5 (aka EcmaScript 3). The Closure Compiler does not support anything that is not compatible with the EcmaScript standard (aka JavaScript). This includes:

- [The `const` keyword](https://developer.mozilla.org/En/Core_JavaScript_1.5_Guide/Constants)
- [Destructuring assignments](https://developer.mozilla.org/en/New_in_javascript_1.7#Destructuring_assignment)
- [for each](https://developer.mozilla.org/en/JavaScript/Reference/Statements/for_each...in)



### Closure Compiler inlined all my strings, which made my code size bigger. Why did it do that?

Most people compare code size by looking at two uncompressed JavaScript files. But that's a misleading way to look at code size, because your JavaScript files should not be served uncompressed. It should be served with gzip compression.

Closure Compiler assumes that you are using gzip compression. If you do not, you should. Configuring your server to [gzip your code](http://developer.yahoo.com/blogs/ydn/posts/2007/07/high_performanc_3/) is one of the most effective and easiest optimizations that you can possibly do. [The gzip algorithm](http://en.wikipedia.org/wiki/Gzip) works by trying to alias sequences of bytes in an optimal way. Aliasing strings manually almost always makes the compressed code size bigger, because it subverts gzip's own algorithm for aliasing. So Closure Compiler will (almost) always inline your strings when it can, because that will make your compressed code smaller.

If you know your client doesn't support gzip, the Java API has an `aliasAllStrings` option on the [CompilerOptions](http://code.google.com/p/closure-compiler/source/browse/src/com/google/javascript/jscomp/CompilerOptions.java) class. Turning this on will make the compiler alias all your strings in the global scope.

### JSC_VAR_MULTIPLY_DECLARED_ERROR occur using the standalone compiler jar but not with the web service

The [web service](http://closure-compiler.appspot.com/#webservice) uses slightly different defaults than the standalone compiler. By default, the web service is more permissive. If the code you are using assumes that it's OK to redeclare global variables, then you will need to using the `--third_party` flag to let the compiler know you don't want these to be treated as errors.

### My code broke when using advanced optimizations! How do I figure out what's wrong?

Use the `--debug` flag combined with the `--formatting=PRETTY_PRINT` flag.

When you're using the `--debug` flag, Closure Compiler will rename all your symbols, but it will give them _longer_ names so that you can figure out what the original name was.

If you are using `ADVANCED` optimizations, you may also want to disable the type-based optimizations with `--use_types_for_optimization false`. Incorrect type optimizations can cause the compiler to make invalid assumptions.

### Some of my properties are getting renamed, but some aren't. Why?

As of the 20150315 release, Closure Compiler will try to use type information to distinguish properties on one type from properties on another type. If you do not want the renaming to use type information, you may disable the type based optimizations using the `--use_types_for_optimization false` flag.

We also wrote a series of blog posts that describes how the renaming algorithm works in more technical detail.

- [Part 1](http://closuretools.blogspot.com/2011/01/property-by-any-other-name-part-1.html)
- [Part 2](http://closuretools.blogspot.com/2011/01/property-by-any-other-name-part-2.html)
- [Part 3](http://closuretools.blogspot.com/2011/01/property-by-any-other-name-part-3.html)

It may be helpful to disable these optimizations to debug errors. Incorrect type information may cause the compiler to make invalid assumptions. This is especially easy to do with event handler arguments. As an example:

```JavaScript
/**
 * @param {Event} evt should really be a goog.events.BrowserEvent
 *   goog.events.BrowserEvent does not inherit from Event and so the
 *   compiler may rename the stopPropagation method in a breaking fashion
 */
function myEventHandler(evt) {
  evt.stopPropagation();
}

goog.events.listen(document.getElementById('myanchor'), myEventHandler, false);
```

### When using type-checking, sometimes Closure Compiler doesn't warn me about missing properties. Why not?

[This blog post](http://closuretools.blogspot.com/2011/01/property-by-any-other-name-part-3.html) describes how to turn the missing properties check on, and what it can do for you.

It's important to keep in mind that Closure Compiler's type system is an optional type system. It doesn't require that you annotate every function with types. And it will only tell you that a type is wrong if it knows for sure that the type is wrong. It won't complain if it simply can't figure out the type (as the java compiler would).

This has some interesting corollaries. The one that gotchas people the most is when you write code like this 

    element.contentWindow.focus();

Not all elements have "contentWindow" defined on them. Only HTMLIframeElements do. Closure Compiler will not complain about this, because it only warns about missing properties if it knows that property 'cannot possibly' be defined. (The java compiler will do the reverse, and warn about missing properties if it knows that the property 'may not' be defined).  This approach was chosen as many JS APIs (like the DOM) are designed so that the properties on an object are only truly knowable at run-time.

It is possible to get stricter property check on specific types by annotating them with [`@struct`](https://github.com/google/closure-compiler/wiki/@struct-and-@dict-Annotations).   The experimental [New Type Inferrence](https://github.com/google/closure-compiler/wiki/Using-NTI-(new-type-inference)) implements a 'may not be defined' check.

## Using Closure Compiler

### What are the recommended Java VM command-line options?

While the compiler will correctly run with `java -jar compiler.jar` there are two ways to significantly improve the compilation time for most jobs. Try one of these command-lines:

1. `java -client -d32 -jar compiler.jar`
1. `java -server -XX:+TieredCompilation -jar compiler.jar`

### How do I use Advanced Optimizations?

See the [official tutorial](http://code.google.com/closure/compiler/docs/api-tutorial3.html).

### I want to use Advanced Optimizations, but { jQuery, YUI, Underscore, Prototype, JS Library Foo } does not work with them. What do I do?

[Externs](http://code.google.com/closure/compiler/docs/api-tutorial3.html#externs) are a good solution to this. If you come from a C or C++ background, you can think of an externs file as a header file. It declares the API for code outside the main compilation process. 

You split the code into 3 pieces:
- The library code
- An externs file for the API of the library code
- The application code

Then, you can compile the application code with Advanced Optimizations by passing the externs file with the --externs flag.

For common JS libraries, it's likely that someone has already written an externs file that you can use (see below).

### Do I need to write type annotations to take advantage of Advanced Optimizations?

No. 

The advanced optimizations don't really use type information at all.

The big wins in advanced optimizations come from [externs and exports](http://code.google.com/closure/compiler/docs/api-tutorial3.html#removal). Closure Compiler will remove code that isn't used or exported.

### When using Advanced Optimizations, Closure Compiler adds new variables to the global scope. How do I make sure my variables don't collide with other scripts on the page?

Closure Compiler's advanced optimizations mode assumes that it's ok to add new variables to the global scope.

In JavaScript, it's often standard practice to wrap your code in an anonymous function, so that variables don't pollute the global scope. Closure Compiler has an `--output_wrapper` flag for exactly this purpose. Invoking it as `--output_wrapper "(function() {%output%}).call(window);"` will wrap your code in an anonymous function at compile-time.

Do not manually wrap your code in an immediately executed function, because the optimization passes may inline the call.

### How do I call Closure Compiler from the Java API?

There's no official tutorial for the Java API. There's a short code snippet on [this blog](http://blog.bolinfest.com/2009/11/calling-closure-compiler-from-java.html) that, in our opinion, gives a nice 1-minute demo on how to do this.

If you're looking at Closure Compiler's source code, [AbstractCommandLineRunner.java](http://code.google.com/p/closure-compiler/source/browse/src/com/google/javascript/jscomp/AbstractCommandLineRunner.java) and [CommandLineRunner.java](http://code.google.com/p/closure-compiler/source/browse/src/com/google/javascript/jscomp/CommandLineRunner.java) do the heavy-lifting of translating command-line flags into Java API calls, so you could find some inspiration there. For a comprehensive reference on the Java API, we check our [JavaDoc](http://closure-compiler.googlecode.com/git/javadoc/index.html) into Git, for easy browsing. It will always reflect the version at trunk.

Notice that Closure Compiler uses threads. Many Java implementations (like [Google App Engine](http://code.google.com/appengine/), for example) do not support threads, so you may need to call the [disableThreads](http://closure-compiler.googlecode.com/git/javadoc/com/google/javascript/jscomp/Compiler.html#disableThreads()) method.

### How do I write an externs file?

Externs files define APIs outside (or "external") to your code. the externs file should define all external variables, types, and properties that you intend to interact with. For example, Closure Compiler doesn't know about the browser APIs for interacting with the window and the DOM. Instead, we ship Closure Compiler with a default set of externs files that defines the browser APIs. This default set is probably the best example of how to write an externs file. As you can see, it defines global variables like `window`, types like `Range`, and properties like `Window.prototype.alert`.

https://github.com/google/closure-compiler/tree/master/externs

If you need an externs file for a common API (like the Google Maps API), first try asking on [closure-compiler-discuss](http://groups.google.com/group/closure-compiler-discuss) whether someone has already written one. Many people have contributed externs for common JS libraries in our contrib directory.

https://github.com/google/closure-compiler/tree/master/contrib/externs

It is also possible to automatically generate externs for a common library by running that library in a JS interpreter, and then traversing its objects with a little bit of JS code. Some people have even wrote web services to do this for you.

http://blog.dotnetwise.com/2009/11/closure-compiler-externs-extractor.html

Notice that automatically-generated externs files lack type annotations. They will also miss out on parameters and return values of the external API.

You can find more information about externs files in [the official documentation](http://code.google.com/closure/compiler/docs/api-tutorial3.html#externs).

### How do I add closure-compiler functionality to my project?

The easiest way to include closure-compiler in your project is to use the artifacts deployed to [[Maven]].  You can also take the closure-compiler jar file and include it in your project.  It contains all of the code and dependencies.  If you do include the actual jar file be aware that the you may have version mismatches with the bundled libraries (args4j, protobuf, guava, etc.).

### What version of Closure Compiler should I use?

Closure Compiler does regular releases of a built JAR file. You can find them on the [[Binary Downloads]] page.

We publish these JARs to the [[Maven|Maven]] repository.

It is also safe to checkout Git head and use that. See the [README](https://github.com/google/closure-compiler/tree/master/README.md) for instructions on how to build it. Closure Compiler team believes in continuous integration: the code in Git should always work. Several of our users re-build Closure Compiler from head multiple times a day, and run all their JS tests against it. This means we can usually find regressions within a few hours of when they're checked in. Regressions are almost always rolled back immediately.

### Source Maps and sourceMappingURL
Closure Compiler provides the `--create_source_map` flag to generate a [JavaScript source map](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/). However, the compiler does not automatically add the sourceMappingUrl comment to the output file.

Adding the sourceMappingUrl comment can be done with the `--output_wrapper` flag:

```
--output_wrapper "%output%\n//# sourceMappingURL=output.js.map"
```

## Contributions

### How do I submit a patch?

Check out our [[guide for contributors|Contributors]].

### How do I submit a bug report?

Go to the [Issue Tracker](https://github.com/google/closure-compiler/issues) and search to see if your problem has already been filed. If it has not, please [file a new one](https://github.com/google/closure-compiler/issues/new). We try to respond to new bugs within a day. Feature requests may take longer to get a response.

### How do I submit a feature request for a new type of optimization?

If you want us to add new optimizations that will make your code smaller or faster, you can also file that as a feature request in the issue tracker. Please label it with "enhancement". We are more than happy to add more optimizations because they do help lots of people.

However, keep in mind that there are infinitely many optimizations we could perform. Each extra optimization adds a maintenance cost and makes the compiler run more slowly. This is not an exaggeration: the number really is infinite. Wikipedia calls this the ["Full Employment Theorem for Compiler Writers"](http://en.wikipedia.org/wiki/Full_employment_theorem), and it basically says that even if we had infinite engineers, you would always be able to give them more work.

If you want an optimization idea implemented, please provide details on why that optimization is important so that we can prioritize it (e.g., "This optimization would reduce JS file X by Y%"). Sometimes we will close an optimization request as invalid, because we believe that the benefit would be too small to make it worthwhile, or because it optimizes a code pattern that shows up very rarely in real code.

A few years ago, Kevin Bourrillion wrote [a thoughtful explanation](https://plus.google.com/113026104107031516488/posts/ZRdtjTL1MpM) of the Guava project's policy for accepting feature requests and patches. Closure Compiler has a more liberal patch-acceptance policy than Guava, but many of the ideas in that post are relevant.

## Design of Closure Compiler

### How do I find out more about how Closure Compiler works?

See our [[Design Documents|Design-Documents]].

### Is there a specification for the JSDoc type language?

The Closure Compiler type language tries to adhere as closely as possible to the [EcmaScript 4 draft spec](http://wiki.ecmascript.org/doku.php?id=spec:spec), which contains a rigorous grammar for the type language, and a description of the intended semantics.

We also have a more informal tutorial on [how to write type annotations](http://code.google.com/closure/compiler/docs/js-for-compiler.html#types).

If you look at Closure Library, you may see some type annotations that do not fit the grammar exactly as specified, but get parsed correctly anyway. This is usually intentional. The EcmaScript 4 draft was in flux as we were implementing it, and so there are some legacy type syntaxes that we still support. (This mostly involves the order of certain operators, and the operator used for union types).

### Can you add alternative syntax X to your JSDoc type language?

If X adds a way to do something that wasn't possible before, then you should propose your idea on [closure-compiler-discuss](http://groups.google.com/group/closure-compiler-discuss).

If X is already possible some other way, then 99% of the time, the answer is no. 

There are many tools that need to parse our JSDoc type language. When you add a new syntax, that means all those tools need to be updated. The documentation needs to be updated, and every JavaScript developer needs to be taught that the syntaxes are equivalent. It becomes harder to write new tools, because the language is now more complicated.

If this new syntax doesn't add any real value, then we really can't justify that cost.

### I'm writing a Firefox extension. It only needs to work on the bleeding-edge version of Firefox. I don't care if it doesn't work in other browsers. Could we add a `--platform=FIREFOX` flag that lets me use the snazzy JavaScript features introduced in  [JavaScript 1.7](https://developer.mozilla.org/en/New_in_javascript_1.7)?

In general, no. There are two reasons for this.

The first is that we're lazy. In the past, Closure Compiler passes were mostly written by Google engineers in their volunteer time. The development cycle went like, "Hey! My app is running slow, but I could speed it up if I added this optimization. I can write it in an afternoon and then go back to my full-time job." Most developers are not experts on JavaScript. So it was pretty important to us that it was easy for developers to write new passes without understanding the two-dozen variants of JavaScript.

The second is that we really like sharing code. Suppose I write a JavaScript class for my Firefox extension that uses Firefox-only keywords. A couple months later, my friend Kushal likes it, and wants to use it in his web app. His web app has to work on Internet Explorer. But trying to untangle all the [getters and setters](https://developer.mozilla.org/En/Core_JavaScript_1.5_Guide:Creating_New_Objects:Defining_Getters_and_Setters) takes way too long. He gives up when he realizes that it's easier for him to rewrite the code from scratch. This makes both of us unhappy. We find that everyone's happier if code works cross-browser, and it's easy to share.

### My question isn't covered on this list!

Try asking on [closure-compiler-discuss](http://groups.google.com/group/closure-compiler-discuss).