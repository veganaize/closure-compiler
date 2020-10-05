# `import.meta` support

Because the compiler always generates a single output script without ES modules, it's not really possible to emulate the behavior of `import.meta`. Instead the compiler reports an error if it is used.