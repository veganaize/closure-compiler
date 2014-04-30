# Guidelines for submitting patches to Closure Compiler

### So! You Want to Submit a Patch?

Getting patches into Closure Compiler should be relatively pain-free. We are still ramping up the process for accepting patches, so it may be slow at first. And if it is, you should complain loudly. Complaining is the only way to ensure that things get better. 

This page should help guide you through the submitting process. We assume the reader already knows how to complain.

### Lifecycle of a Patch

To submit a patch, create a bug entry on the [issues list](http://code.google.com/p/closure-compiler/issues/list) and attach your diff. You will need to sign a [Contributor License Agreement](http://code.google.com/p/closure-compiler/source/browse/CONTRIBUTORS) if you haven't done so. 

One of our committers will accept the patch by marking it as "Accepted" in the bug tracker. They may ask you additional questions about the change, or request improvements. Once the first committer is satisfied, he or she will ask a second committer to review the change and sign off on it.

If the patch is substantial, we may ask you to upload the patch to Rietveld to make comments. Download [upload.py](http://code.google.com/p/rietveld/wiki/UploadPyUsage). In a typical git client, you will usually invoke it like 

{{{upload.py --rev master --cc closure-compiler-reviews@googlegroups.com --send_mail}}}

to upload your change.

Some changes, particularly changes that add new optimizations, may require extensive integration testing to ensure they don't break under unforeseen edge cases. Fortunately, most of our committers work on large, proprietary JavaScript codebases with lots of integration tests. We may test your change against a few of those projects before we commit it. So if takes a day or two for us to commit your change after it has been reviewed, we may just be running integration tests on it first.

Changes are automatically mirrored between code.google.com's Git repository, and Google's internal Perforce repository. The committer may submit your change to either Git or perforce.


### How to Get Your Patch Accepted

A few pointers on how to get your patch accepted:
- Run `ant test` and ensure your patch passes all unit tests.
- Write a unit test that demonstrates your patch. Tests are the best way to ensure that future contributors do not break your code accidentally.
- Follow the coding conventions (below)

### Coding Conventions

For the most part, we try to follow [Sun's Coding Conventions for the Java Programming Language](http://java.sun.com/docs/codeconv/html/CodeConvTOC.doc.html), with a few exceptions.

- We use 2 space indents for blocks. We never use tabs.
- All files must begin with the standard Apache License 2.0 header.
- Lines of code may not exceed 80 characters. There are a few exceptions to this rule, mostly due to syntax that can't be broken across lines (imports, URLs, etc.)

When in doubt, please be consistent with the other code in the codebase. We realize that style rules are largely arbitrary, and are not necessarily better or worse than what you may be used to. We have style rules mostly because consistently formatted code is easier to read, not because we favor one particular style over another.