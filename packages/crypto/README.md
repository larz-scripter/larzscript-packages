# lz-crypto

Real SHA-256 + HMAC-SHA256, in pure Larzscript (FIPS 180-4). Runs a self-test
against the standard NIST vectors at import time. `larzscript larzpkg.lz install crypto`.

```
import "crypto" as crypto
crypto.sha256("abc")            # "ba7816bf..."
crypto.hmac("secret", "body")   # HMAC-SHA256 hex digest
```
