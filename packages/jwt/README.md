# lz-jwt

Signed JSON tokens (header.payload.signature, base64url), via `crypto` +
`json` + `base64`. Hosted-only. `larzscript larzpkg.lz install jwt`.

```
import "jwt" as jwt
let token = jwt.sign({"sub": "user1"}, "secret")
let payload = jwt.verify(token, "secret")
```
