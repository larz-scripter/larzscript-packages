# lz-totp

RFC 6238 time-based one-time codes (HMAC-SHA256 variant), via `crypto`.
Hosted-only. `larzscript larzpkg.lz install totp`.

```
import "totp" as totp
let code = totp.generate(secret)
totp.verify(secret, code)
```
