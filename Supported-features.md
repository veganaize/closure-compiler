| Feature           | Language Version | Support |
| ----------------- | ---------------- | ------- |
| ES3 keywords as identifiers | `ES5` | yes |
| getters & setters |            `ES5` | Yes, but cannot be transpiled down to `ES3`. Negatively impact optimizations. |
| reserved words as properties | `ES5` | yes |
| string continuation | `ES5` | yes |
| trailing comma    |            `ES5` | yes |
| arrow function | `ES6` | yes |
| binary literal | `ES6` | yes |
| octal literal | `ES6` | yes |
| block-scoped function declaration | `ES6` | yes |
| class | `ES6` | yes |
| class extends | `ES6` | yes |
| class getters / setters | `ES6` | Yes, but cannot be transpiled down to `ES3`. Negatively impact optimizations. |
| computed properties | `ES6` | yes |
| const declaration | `ES6` | yes |
| let declaration | `ES6` | yes |
| default parameter | `ES6` | yes |
| rest parameter | `ES6` | yes |
| array destructuring | `ES6` | yes |
| array pattern rest | `ES6` | yes |
| object destructuring | `ES6` | yes |
| extended object literal | `ES6` | yes |
| member declaration | `ES6` | yes |
| for-of loop | `ES6` | yes |
| generators | `ES6` | yes |
| `new.target` | `ES6` | yes |
| `RegExp` flag 'u' | `ES6` | recognized in input, but cannot be transpiled |
| `RegExp` flag 'y' | `ES6` | recognized in input, but cannot be transpiled |
| string template literals | `ES6` | yes |
| ES modules | `ES6` | yes in input, but output is one large script, possibly broken into chunks |
| exponent operators | `ES7` | yes |
| async functions | `ES8` | yes |
| trailing comma in param list | `ES9` | yes |
| spread into object literal | `ES_2018` | yes |
| object pattern rest | `ES_2018` | yes |
| async generator functions | `ES_2018` | yes |
| for-await-of loop | `ES_2018` | yes |
| `RegExp` flag 's' | `ES_2018` | recognized in input, but cannot be transpiled |
| `RegExp` lookbehind | `ES_2019` | recognized in input, but cannot be transpiled |
| `RegExp` named groups | `ES_2018` | recognized in input, but cannot be transpiled |
| `RegExp` unicode property escape | `ES_2018` | yes |
| Unescaped Unicode line or paragraph separator | `ES_2019` | yes |
| Optional catch binding | `ES_2019` | yes |
| Dynamic module import | `ES_UNSUPPORTED` | recognized in input, but cannot be output or transpiled |
| `import.meta` | `ES_NEXT` | recognized in input, but cannot be output or transpiled |
| Nullish coalescing operator `??` | `ES_UNSUPPORTED` | work in progress |
| Optional chaining operator `?.` | none | work to start soon |

