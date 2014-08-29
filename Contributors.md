# Guidelines for submitting pull requests to Closure Compiler

### So! You Want to Submit a Pull Request?

Getting pull requests into Closure Compiler should be relatively pain-free. We are still ramping up the process for accepting pull requests, so it may be slow at first. And if it is, you should complain loudly. Complaining is the only way to ensure that things get better. 

This page should help guide you through the submitting process. We assume the reader already knows how to complain.

### Lifecycle of a Patch

To submit a pull request, you will need to sign a [Contributor License Agreement](https://github.com/google/closure-compiler/blob/master/CONTRIBUTORS) if you haven't done so. 

One of our committers may ask you additional questions about the change, or request improvements. Once the first committer is satisfied, he or she will ask a second committer to review the change and sign off on it.

Some changes, particularly changes that add new optimizations, may require extensive integration testing to ensure they don't break under unforeseen edge cases. Fortunately, most of our committers work on large, proprietary JavaScript codebases with lots of integration tests. We may test your change against a few of those projects before we commit it. So if takes a day or two for us to commit your change after it has been reviewed, we may just be running integration tests on it first.

Changes are automatically mirrored between github, and Google's internal Perforce repository. The committer may submit your change to either Git or perforce.


### How to Get Your Pull Request Accepted

A few pointers on how to get your pull request accepted:
- Run `ant test` and ensure your patch passes all unit tests.
- Write a unit test that demonstrates your patch. Tests are the best way to ensure that future contributors do not break your code accidentally.
- Follow the coding conventions (below)

### Coding Conventions

For the most part, we try to follow [Sun's Coding Conventions for the Java Programming Language](http://java.sun.com/docs/codeconv/html/CodeConvTOC.doc.html), with a few exceptions.

- We use 2 space indents for blocks. We never use tabs.
- All files must begin with the standard Apache License 2.0 header.
- Lines of code may not exceed 80 characters. There are a few exceptions to this rule, mostly due to syntax that can't be broken across lines (imports, URLs, etc.)

When in doubt, please be consistent with the other code in the codebase. We realize that style rules are largely arbitrary, and are not necessarily better or worse than what you may be used to. We have style rules mostly because consistently formatted code is easier to read, not because we favor one particular style over another.