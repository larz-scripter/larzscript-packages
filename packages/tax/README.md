# lz-tax

Flat-rate and progressive-bracket tax calculators, Money-aware.
`larzscript pkg install tax`.

```
import "tax" as tax
tax.flat($50000.00, 0.2)
tax.progressive($55000.00, [[$0.00, $10000.00, 0.1], [$10000.00, nil, 0.2]])
```
