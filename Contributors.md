# Guidelines for submitting pull requests to Closure Compiler

### So! You Want to Submit a Pull Request?

Getting pull requests into Closure Compiler should be relatively pain-free. We are still ramping up the process for accepting pull requests, so it may be slow at first. And if it is, you should complain loudly. Complaining is the only way to ensure that things get better. 

This page should help guide you through the submitting process. We assume the reader already knows how to complain.

### Lifecycle of a Pull Request

To submit a pull request, you will need to sign a [Contributor License Agreement](https://github.com/google/closure-compiler#submitting-patches) if you haven't done so. 

One of our committers may ask you additional questions about the change, or request improvements. Rather than putting those improvements in a separate commit, please "squash" them into a single commit. This will make it easier for us to mirror the change into Google's internal repository. Once the first committer is satisfied, they may ask a second committer to review the change and sign off on it.

Some changes, particularly changes that add new optimizations, may require extensive integration testing to ensure they don't break under unforeseen edge cases. Fortunately, most of our committers work on large, proprietary JavaScript codebases with lots of integration tests. We may test your change against a few of those projects before we commit it. So if takes a day or two for us to commit your change after it has been reviewed, we may just be running integration tests on it first.

Changes are automatically mirrored between Github, and Google's internal repository. Once your change is ready to be merged, the committer will generally submit it to Google's internal repository first, and then it will be mirrored to the master branch on Github, usually within a day or two.

### How to Get Your Pull Request Accepted

A few pointers on how to get your pull request accepted:
- Run `mvn test` and ensure your patch passes all unit tests.
- Write a unit test, or possibly an integration test, that demonstrates your patch. Tests are the best way to ensure that future contributors do not break your code accidentally.
- Follow the coding conventions (below)

### Coding Conventions

For the most part, we try to follow [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html). Please note:

- All files must begin with the standard Apache License 2.0 header.
- We have found the [`google-java-format`](https://github.com/google/google-java-format) tool very helpful for fixing up formatting, which takes care of using correct indentation, wrapping at 100 characters, etc. We recommend only reformatting the lines you've changed, so that your pull request isn't filled with lots of unrelated formatting changes. See the [`google-java-format-diff`](https://github.com/google/google-java-format/blob/master/scripts/google-java-format-diff.py) script.

When in doubt, please be consistent with the other code in the codebase. We realize that style rules are largely arbitrary, and are not necessarily better or worse than what you may be used to. We have style rules mostly because consistently formatted code is easier to read, not because we favor one particular style over another.