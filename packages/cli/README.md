# lz-cli

build CLI tools with subcommands and help. `larzscript pkg install cli`.

```
import "cli" as cli
let app = cli.new("tool", "desc")
cli.command(app, "run", "run it", fn(a) { print("go") })
cli.run(app, args)
```
