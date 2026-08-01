# lz-validate

Input validation and hygiene checks - email/URL format, path traversal,
SQLi/XSS pattern heuristics (defense-in-depth, not a guarantee - always
use parameterized queries and proper output encoding).
`larzscript larzpkg.lz install validate`.

```
import "validate" as validate
validate.is_email("a@b.com")
validate.looks_like_sqli(input)
```
