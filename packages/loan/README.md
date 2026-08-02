# lz-loan

Interest and amortization calculations, Money-aware.
`larzscript pkg install loan`.

```
import "loan" as loan
loan.simple_interest($1000.00, 0.05, 2)
let schedule = loan.amortize($10000.00, 0.06, 12)
```
