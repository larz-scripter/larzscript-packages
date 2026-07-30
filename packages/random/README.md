# lz-random

a small seeded PRNG (deterministic, non-crypto). `larzscript larzpkg.lz install random`.

```
import "random" as r
r.seed(42); print(r.randint(1, 6))
```
