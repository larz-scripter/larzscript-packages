# lz-args

Simple CLI argument parsing for Larzscript. `larzscript pkg install args`.

```
import "args" as args
let p = args.parse(args_global)      # pass the global `args`
args.flag(p, "name", "default")      # --name=value
p["flags"]["verbose"]                # --verbose -> true
p["positional"]                      # the rest
```
