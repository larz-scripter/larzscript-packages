# lz-budget

Category-based budget tracking on the Money type.
`larzscript larzpkg.lz install budget`.

```
import "budget" as budget
let b = budget.new()
budget.set_limit(b, "groceries", $300.00)
budget.spend(b, "groceries", $45.00, "weekly shop")
budget.status(b, "groceries")
```
