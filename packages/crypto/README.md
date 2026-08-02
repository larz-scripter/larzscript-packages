# lz-crypto

Real SHA-256 + HMAC-SHA256, in pure Larzscript (FIPS 180-4). Runs a self-test
against the standard NIST vectors at import time. Also has `pbkdf2` (a
password KDF - note each iteration is ~0.5s on this interpreter, so real
production iteration counts aren't practical here) and `hmac_bytes`/
`hmac_raw`/`str_to_bytes` for callers that need raw bytes (used by `totp`/
`jwt`). `larzscript pkg install crypto`.

```
import "crypto" as crypto
crypto.sha256("abc")            # "ba7816bf..."
crypto.hmac("secret", "body")   # HMAC-SHA256 hex digest
crypto.pbkdf2("password", "salt", 10)
```
