Last update: 2020-01-09

See also https://github.com/google/closure-compiler/issues/2899

| Feature                                       | Language Version | Support                                                                       |
| --------------------------------------------- | ---------------- | ----------------------------------------------------------------------------- |
| ES3 keywords as identifiers                   | `ES5`            | yes                                                                           |
| getters & setters                             | `ES5`            | Yes, but cannot be transpiled down to `ES3`. Negatively impact optimizations. |
| reserved words as properties                  | `ES5`            | yes                                                                           |
| string continuation                           | `ES5`            | yes                                                                           |
| trailing comma                                | `ES5`            | yes                                                                           |
| array destructuring                           | `ES6`            | yes                                                                           |
| `Array.from`                                  | `ES6`            | polyfill                                                                      |
| `Array.of`                                    | `ES6`            | polyfill                                                                      |
| array pattern rest                            | `ES6`            | yes                                                                           |
| `Array.prototype.copyWithin`                  | `ES6`            | polyfill                                                                      |
| `Array.prototype.entries`                     | `ES6`            | polyfill                                                                      |
| `Array.prototype.fill`                        | `ES6`            | polyfill                                                                      |
| `Array.prototype.find`                        | `ES6`            | polyfill                                                                      |
| `Array.prototype.findIndex`                   | `ES6`            | polyfill                                                                      |
| `Array.prototype.keys`                        | `ES6`            | polyfill                                                                      |
| arrow function                                | `ES6`            | yes                                                                           |
| binary literal                                | `ES6`            | yes                                                                           |
| block-scoped function declaration             | `ES6`            | yes                                                                           |
| class                                         | `ES6`            | yes                                                                           |
| class extends                                 | `ES6`            | yes                                                                           |
| class getters / setters                       | `ES6`            | Yes, but cannot be transpiled down to `ES3`. Negatively impact optimizations. |
| computed properties                           | `ES6`            | yes                                                                           |
| const declaration                             | `ES6`            | yes                                                                           |
| default parameter                             | `ES6`            | yes                                                                           |
| ES modules                                    | `ES6`            | yes in input, but output is one large script, possibly broken into chunks     |
| extended object literal                       | `ES6`            | yes                                                                           |
| for-of loop                                   | `ES6`            | yes                                                                           |
| generators                                    | `ES6`            | yes                                                                           |
| let declaration                               | `ES6`            | yes                                                                           |
| `Map`                                         | `ES6`            | polyfill                                                                      |
| `Math.acosh`                                  | `ES6`            | polyfill                                                                      |
| `Math.asinh`                                  | `ES6`            | polyfill                                                                      |
| `Math.atanh`                                  | `ES6`            | polyfill                                                                      |
| `math.cbrt`                                   | `ES6`            | polyfill                                                                      |
| `Math.clz32`                                  | `ES6`            | polyfill                                                                      |
| `Math.cosh`                                   | `ES6`            | polyfill                                                                      |
| `Math.expm1`                                  | `ES6`            | polyfill                                                                      |
| `Math.fround`                                 | `ES6`            | polyfill                                                                      |
| `Math.hypot`                                  | `ES6`            | polyfill                                                                      |
| `Math.imul`                                   | `ES6`            | polyfill                                                                      |
| `Math.log10`                                  | `ES6`            | polyfill                                                                      |
| `Math.log1p`                                  | `ES6`            | polyfill                                                                      |
| `Math.log2`                                   | `ES6`            | polyfill                                                                      |
| `Math.sign`                                   | `ES6`            | polyfill                                                                      |
| `Math.sinh`                                   | `ES6`            | polyfill                                                                      |
| `Math.tanh`                                   | `ES6`            | polyfill                                                                      |
| `Math.trunc`                                  | `ES6`            | polyfill                                                                      |
| member declaration                            | `ES6`            | yes                                                                           |
| `new.target`                                  | `ES6`            | yes                                                                           |
| `Number.EPSILON`                              | `ES6`            | polyfill                                                                      |
| `Number.isFinite`                             | `ES6`            | polyfill                                                                      |
| `Number.isInteger`                            | `ES6`            | polyfill                                                                      |
| `Number.isNaN`                                | `ES6`            | polyfill                                                                      |
| `Number.isSafeInteger`                        | `ES6`            | polyfill                                                                      |
| `Number.MAX_SAFE_INTEGER`                     | `ES6`            | polyfill                                                                      |
| `Number.MIN_SAFE_INTEGER`                     | `ES6`            | polyfill                                                                      |
| `Number.parseFloat`                           | `ES6`            | polyfill                                                                      |
| `Number.parseInt`                             | `ES6`            | polyfill                                                                      |
| `Object.assign`                               | `ES6`            | polyfill                                                                      |
| object destructuring                          | `ES6`            | yes                                                                           |
| `Object.is`                                   | `ES6`            | polyfill                                                                      |
| `Object.setPrototypeOf`                       | `ES6`            | polyfill                                                                      |
| octal literal                                 | `ES6`            | yes                                                                           |
| `Promise`                                     | `ES6`            | polyfill                                                                      |
| `Reflect.apply`                               | `ES6`            | polyfill                                                                      |
| `Reflect.construct`                           | `ES6`            | polyfill                                                                      |
| `Reflect.defineProperty`                      | `ES6`            | polyfill                                                                      |
| `Reflect.deleteProperty`                      | `ES6`            | polyfill                                                                      |
| `Reflect.get`                                 | `ES6`            | polyfill                                                                      |
| `Reflect.getOwnPropertyDescriptor`            | `ES6`            | polyfill                                                                      |
| `Reflect.getPrototypeOf`                      | `ES6`            | polyfill                                                                      |
| `Reflect.has`                                 | `ES6`            | polyfill                                                                      |
| `Reflect.isExtensible`                        | `ES6`            | polyfill                                                                      |
| `Reflect.ownKeys`                             | `ES6`            | polyfill                                                                      |
| `Reflect.preventExtensions`                   | `ES6`            | polyfill                                                                      |
| `Reflect.set`                                 | `ES6`            | polyfill                                                                      |
| `Reflect.setPrototypeOf`                      | `ES6`            | polyfill                                                                      |
| `RegExp` flag 'u'                             | `ES6`            | recognized in input, but cannot be transpiled                                 |
| `RegExp` flag 'y'                             | `ES6`            | recognized in input, but cannot be transpiled                                 |
| rest parameter                                | `ES6`            | yes                                                                           |
| `Set`                                         | `ES6`            | polyfill                                                                      |
| `String.fromCodePoint`                        | `ES6`            | polyfill                                                                      |
| `String.prototype.codePointAt`                | `ES6`            | polyfill                                                                      |
| `String.prototype.endsWith`                   | `ES6`            | polyfill                                                                      |
| `String.prototype.includes`                   | `ES6`            | polyfill                                                                      |
| `String.prototype.repeat`                     | `ES6`            | polyfill                                                                      |
| `String.prototype.startsWith`                 | `ES6`            | polyfill                                                                      |
| string template literals                      | `ES6`            | yes                                                                           |
| `WeakMap`                                     | `ES6`            | polyfill                                                                      |
| `WeakSet`                                     | `ES6`            | polyfill                                                                      |
| exponent operators                            | `ES_2016`        | yes                                                                           |
| `Array.prototype.includes`                    | `ES_2016`        | polyfill                                                                      |
| `Array.prototype.values`                      | `ES_2017`        | polyfill                                                                      |
| async functions                               | `ES_2017`        | yes                                                                           |
| `Object.entries`                              | `ES_2017`        | polyfill                                                                      |
| `Object.getOwnPropertyDescriptors`            | `ES_2017`        | polyfill                                                                      |
| `Object.values`                               | `ES_2017`        | polyfill                                                                      |
| `String.prototype.padEnd`                     | `ES_2017`        | polyfill                                                                      |
| `String.prototype.padStart`                   | `ES_2017`        | polyfill                                                                      |
| `Array.prototype.flat`                        | `ES_2018`        | polyfill                                                                      |
| `Array.prototype.flatmap`                     | `ES_2018`        | polyfill                                                                      |
| async generator functions                     | `ES_2018`        | yes                                                                           |
| for-await-of loop                             | `ES_2018`        | yes                                                                           |
| object pattern rest                           | `ES_2018`        | yes                                                                           |
| `Promise.prototype.finally`                   | `ES_2018`        | polyfill                                                                      |
| `RegExp` flag 's'                             | `ES_2018`        | recognized in input, but cannot be transpiled                                 |
| `RegExp` lookbehind                           | `ES_2018`        | recognized in input, but cannot be transpiled                                 |
| `RegExp` named groups                         | `ES_2018`        | recognized in input, but cannot be transpiled                                 |
| `RegExp` unicode property escape              | `ES_2018`        | yes                                                                           |
| spread into object literal                    | `ES_2018`        | yes                                                                           |
| trailing comma in param list                  | `ES_2018`        | yes                                                                           |
| `Object.fromEntries`                          | `ES_2019`        | polyfill                                                                      |
| Optional catch binding                        | `ES_2019`        | yes                                                                           |
| `String.prototype.trimEnd`                    | `ES_2019`        | polyfill                                                                      |
| `String.prototype.trimLeft`                   | `ES_2019`        | polyfill                                                                      |
| `String.prototype.trimRight`                  | `ES_2019`        | polyfill                                                                      |
| `String.prototype.trimStart`                  | `ES_2019`        | polyfill                                                                      |
| Unescaped Unicode line or paragraph separator | `ES_2019`        | yes                                                                           |
| `globalThis`                                  | `ES_2020`        | polyfill                                                                      |
| `import.meta`                                 | `ES_2020`        | recognized in input, but cannot be output or transpiled                       |
| `Promise.allSettled`                          | `ES_2020`        | polyfill                                                                      |
| `String.prototype.matchAll`                   | `ES_2020`        | polyfill                                                                      |
| Dynamic module import                         | `ES_2020`        | recognized in input, but cannot be output or transpiled                       |
| Nullish coalescing operator `??`              | `ES_2020`        | yes                                                                     |
| Optional chaining operator `?.`               | `ES_2020`        | yes                                                                     |
| BigInt                                        | `ES_2020`        | see [BigInt Support](https://github.com/google/closure-compiler/wiki/BigInt-support) |