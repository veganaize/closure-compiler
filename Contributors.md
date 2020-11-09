# Guidelines for submitting pull requests to Closure Compiler

This page should help guide you through the submitting process.

### [Code of Conduct]

[Code of Conduct]: https://github.com/google/closure-compiler/blob/master/code_of_conduct.md

All interactions within the closure-compiler community are subject our [Code of Conduct].
Please bear that in mind when filing pull requests and responding to comments on them.

### Contributor License Agreement (CLA)

You will need to sign a [Contributor License Agreement](https://github.com/google/closure-compiler#submitting-patches) if you haven't done so. An automated system will check this when you create a pull request. If you haven't done it yet, it will prompt you with instructions on the pull request itself. We may choose not to review the PR until this has been done.

### Lifecycle of a Pull Request

One of our committers may ask you additional questions about the change, or request improvements. Once the first committer is satisfied, they may ask a second committer to review the change and sign off on it.

Some changes, particularly changes that add new optimizations, may require extensive integration testing to ensure they don't break under unforeseen edge cases. Fortunately, most of our committers work on large, proprietary JavaScript codebases with lots of integration tests. We may test your change against a few of those projects before we commit it. So if takes a day or two for us to commit your change after it has been reviewed, we may just be running integration tests on it first.

Changes are automatically mirrored between Github, and Google's internal repository. Once your change is ready to be merged, the committer will generally submit it to Google's internal repository first, and then it will be mirrored to the master branch on Github, usually within a day or two.

### How to Get Your Pull Request Accepted

A few pointers on how to get your pull request accepted:
- Run `bazelisk test //:all` and ensure your patch passes all unit tests.
- Write a unit test, or possibly an integration test, that demonstrates your patch. Tests are the best way to ensure that future contributors do not break your code accidentally.
- Follow the coding conventions (below)

### Coding Conventions

For the most part, we try to follow [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html). Please note:

- All files must begin with the standard Apache License 2.0 header.
- Only Java 8 features can be used for now. We hope to allow features from Java 9 and up, soon. 
- We have found the [`google-java-format`](https://github.com/google/google-java-format) tool very helpful for fixing up formatting, which takes care of using correct indentation, wrapping at 100 characters, etc. We recommend only reformatting the lines you've changed, so that your pull request isn't filled with lots of unrelated formatting changes. See the [`google-java-format-diff`](https://github.com/google/google-java-format/blob/master/scripts/google-java-format-diff.py) script.

When in doubt, please be consistent with the other code in the codebase. We realize that style rules are largely arbitrary, and are not necessarily better or worse than what you may be used to. We have style rules mostly because consistently formatted code is easier to read, not because we favor one particular style over another.