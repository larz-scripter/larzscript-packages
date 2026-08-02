# lz-stack

A LIFO stack. Install:

```
larzscript pkg install stack
```

```
import "stack" as stack
let s = stack.new()
stack.push(s, 1); stack.push(s, 2)
print(stack.pop(s))       # 2
print(stack.peek(s))      # 1
print(stack.is_empty(s))  # false
```
