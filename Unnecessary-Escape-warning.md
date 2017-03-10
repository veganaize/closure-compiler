# What's up with this "Unnecessary escape" warning?

If you're writing a string intended to be used as a regular expression you likely have special RegExp escapes in it, such as `\s` or `\w`. You may see a warning saying something like "Unnecessary escape: '\w' is equivalent to just 'w'". This is because, while `\w` is a valid escape sequence in regular expressions, it isn't in string literals. In a string literal, `\w` just means `w`. To get the RegExp escape `\w` you need `\\w`. [See this jsfiddle link for a demo](https://jsfiddle.net/0dt01m30/).

In some cases, it might be much simpler to write your code using a `/RegExp literal/` instead of a `'string literal'` to remove the need for a lot of backslashes.