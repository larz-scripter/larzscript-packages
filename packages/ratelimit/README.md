# lz-ratelimit

Token-bucket rate limiting in ~20 lines - caller owns the state.
`larzscript pkg install ratelimit`.

```
import "ratelimit" as ratelimit
let buckets = {}
if ratelimit.allow(buckets, "user:42", 10, 1.0) { ... }
```
