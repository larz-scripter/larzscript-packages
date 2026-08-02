# lz-regex

Pattern matching for Larzscript, built on the interpreter's native
`regex_match`/`regex_find`/`regex_replace`/`regex_split` builtins. Install
with the Larzscript package manager:

```
larzscript pkg install regex
```

Then in your program:

```
import "regex" as regex
print(regex.test("[0-9]+", "abc123"))                        # true
print(regex.find("[0-9]+", "abc123"))                         # [3, 6]
print(regex.matched_text("[0-9]+", "abc123"))                  # "123"
print(regex.replace("[^a-z0-9]+", "Hello, World!".lower(), "-"))  # "hello-world-"
print(regex.split("\\s+", "a   b\tc"))                        # ["a", "b", "c"]
```

## Supported syntax

Literal characters, `.` (any char), `^` `$` (anchors), `*` `+` `?` (greedy
quantifiers), `[...]` / `[^...]` character classes with `a-z` ranges, and
`\d \D \w \W \s \S` shorthand classes plus `\`-escaped literals.

**Not supported:** capture groups, alternation (`|`), lookaround. This is a
deliberately small, fast, correct subset for fixed hand-written patterns
(validation, cleanup, slugifying) - not a full PCRE-class engine. Like any
classic backtracking regex it has exponential worst case on pathological
patterns, so don't feed it untrusted patterns.
