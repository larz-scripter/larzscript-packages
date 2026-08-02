# lz-test

A tiny test framework for Larzscript. `larzscript pkg install test`.

```
import "test" as t
t.assert_eq(add(2, 3), 5, "adds")
t.assert(is_valid(x), "x is valid")
t.report()          # summary; exits 1 on failure
```
