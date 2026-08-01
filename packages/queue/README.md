# lz-queue

A FIFO queue. Install:

```
larzscript larzpkg.lz install queue
```

```
import "queue" as queue
let q = queue.new()
queue.enqueue(q, 1); queue.enqueue(q, 2)
print(queue.dequeue(q))   # 1
print(queue.peek(q))      # 2
print(queue.is_empty(q))  # false
```

`dequeue()` is O(n) (a plain list shift) - fine for script-sized queues,
not a ring-buffer for a million-item hot loop.
