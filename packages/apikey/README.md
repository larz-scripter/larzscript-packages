# lz-apikey

Generate and verify opaque API keys - store the hash, never the raw key.
`larzscript larzpkg.lz install apikey`.

```
import "apikey" as apikey
let key = apikey.generate("sk")
let stored = apikey.hash(key)
apikey.verify(supplied_key, stored)
```
