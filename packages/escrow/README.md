# lz-escrow

Hold funds from a payer, release to a payee (or refund) - built entirely on
this language's own `wallet`/`pay` primitives. `larzscript larzpkg.lz install escrow`.

```
import "escrow" as escrow
escrow.open($40.00, buyer, held)
escrow.release($40.00, held, seller)
```
