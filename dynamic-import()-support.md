# dynamic `import()` support

Because the compiler always generates a single output script it's not clear how to make dynamic import provide the features users really want. Any implementation would be a significant rearchitecting of how the compiler works and what output it generates, which we cannot justify prioritizing at this time.

The compiler will report an error if the input code uses dynamic import.
