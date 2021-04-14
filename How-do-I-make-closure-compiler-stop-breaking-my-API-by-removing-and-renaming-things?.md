**The tl;dr here is "use externs". The rest is showing why and how.**

# Why is closure-compiler breaking me by removing globals I define and renaming properties?

Whole-application optimization has proved to be the most effective at achieving output code size small enough to make the extremely feature-rich applications Google produces possible, so closure-compiler is designed with that model in mind.
It works with the assumption that the code you give it is **all** of the code that is going to exist in the web page when the output compiled code runs. The only exception made is for symbols it is specifically told about using externs files.

Based on this assumption, it will freely rename variables and object properties to achieve better output code size, unless they are specifically mentioned in an externs file. Having a global variable named "Foo" in an externs file will prevent the compiler from removing or renaming that global, but it won't prevent it from renaming properties on it or, if it's a class, properties on instances of it.

Unfortunately, there are several cases where this assumption is incorrect.
These include.

1. Your app exchanges JSON objects with a server.

   The server cannot know about any renaming that closure-compiler might do.

2. Your app has multiple compiled pieces that must interoperate.

   e.g. There's a web worker that must exchange messages with the main page.

   The compiler doesn't see the web worker code and the main page code at the
   same time, so it cannot consistently rename 'foo' to 'a' in both cases.

3. Your app uses an API whose definition isn't part of the compilation.

   e.g. When your app runs you always first load fooLibrary.

4. Your JS provides an API that must be used from code that isn't visible to
   the compiler.

   e.g. You're writing a JS library that you intend people to load into their
   web pages and then use from their own JS code.

The solution in all of these cases is to tell closure-compiler about the API
that needs to be accessed by code it cannot see.  The way to do this is with
externs files.

# What is an externs file?

An externs file is just a `.js` file that has the annotation `@externs` in the
file's  `@fileoverview` comment.  The annotation tells closure-compiler to
treat the file differently from the other `.js` files you give it.

NOTE: You can also tell the compiler that a `.js` file should be treated as
externs by passing it to the compiler with the `--externs` flag instead of the
`--js` flag, but this is no longer the preferred way to do things.

Inside of the externs file are definitions of global variables and types
(a.k.a. APIs) that the compiler needs to know about and leave unchanged in the
output code.

# What does closure-compiler do with the externs files it reads?

closure-compiler keeps the externs files it reads separate from the normal JS
files in its memory.  It never includes the contents of the externs files in
its output in any form.  Instead, it parses the externs file looking for global
variable definitions and type definitions and records all of them as being
effectively "untouchable".

1. The compiler will not rename global variables defined in externs files.
1. The compiler will not remove definitions of global variables defined in externs files.
1. The compiler will not rename properties with names that appear in externs
   files, unless it can 100% prove that some usage it sees in the normal JS
   code cannot possibly be referring to logically the same property.

# How do I keep closure-compiler from breaking my code that uses JS library X?

If X is a widely used library, you can hopefully find that someone has already
created an externs file for it.  One may even exist in the closure-compiler
repository
(https://github.com/google/closure-compiler/tree/master/contrib/externs).

If you aren't so lucky, you'll need to define your own.

Either way, you use it by just including that file on the closure-compiler
command line along with your other source files.

# How do I use an externs file to protect my API?

**Although we're specifically talking about a web worker in this example, this
answer applies to all of the cases where an API is accessed outside of JS code
that is present in a single compilation.**

Suppose your application is in 2 pieces that get compiled separately.
We'll call them the "app" part and the "worker" part.
The app part gets loaded into a web page. The worker part gets loaded into a web worker.
The 2 parts communicate with each other by passing messages.
These messages use an API that must not be modified by the compiler, or the
communication will break.

Here's an example externs file to define this API.

```javascript
/**
 * @fileoverview Defines the API used in my app to communicate between the main
 *     page and the web worker.
 *
 * @externs
 */

/**
 * Requests made to the web worker will look like this.
 * @record
 */
class WorkerRequest {
  constructor() {
    /** @type {number} */
    this.requestId;
    /** @type {string} */
    this.action;
  }
}

/**
 * The worker will send back responses that look like this.
 * @record
 */
class WorkerResponse {
  constructor() {
    /** @type {number} */
    this.requestId;
    /** @type {string} */
    this.action;
    /** @type {number} */
    this.result;
  }
}
```

You include this file on the closure-compiler command line when compiling both
your app code and your worker code.

```shell
$ closure-compiler --compilation_level ADVANCED \
    --js_output_file app_compiled.js --js myexterns.js --js app.js
$ closure-compiler --compilation_level ADVANCED \
    --js_output_file worker_compiled.js --js myexterns.js --js worker.js
```

You then use can these definitions in your app and worker JS code.
Here's how it might look in the application code.

```javascript
// app part JS file
/**
 * Send a request to the worker.
 *
 * @param {!WorkerRequest} request
 * @return {!Promise<!WorkerResponse>}
 */
async function sendWorkerRequest(request) {
  // code here to send the request object to the worker and arrange to await
  // the response
}

// somewhere later we use the method
async function doSomethingNifty() {
  // The `requestId` an `action` properties here will not be renamed, because
  // the compiler recognizes them as belonging to a record type defined in an
  // externs file.
  const response = await sendWorkerRequest(
      { requestId: getNextRequestId(), action: 'doSomethingNifty' })
  // handle the response
}

// Properties that have the same name, but are provably not related to
// properties in externs can still be renamed due to property disambiguation
// done in ADVANCED mode.
class SomeInternalClass {
  constructor(action) {
    // As long as no instance of SomeInternalClass gets used in a way that the
    // compiler thinks might lead to it being used as if it were a WorkerRequest or
    // WorkerResponse, the compiler will still be able to rename this 'action'
    // property.
    /** @type {string} */
    this.action = action;
  }

  // remainder of this class
}
```
