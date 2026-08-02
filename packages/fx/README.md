# lz-fx

Currency conversion with a settable rate table, fully offline.
`larzscript pkg install fx`.

```
import "fx" as fx
fx.set_rate("USD", "NGN", 1550.0)
fx.convert($10.00, "USD", "NGN")
```
