# lz-crdt

Conflict-free replicated data types: state that merges from multiple
diverged replicas back into one consistent value, with no coordination and
no "last write wins the whole object" data loss. Every merge here is
commutative, associative and idempotent. Directly relevant to a
money-native language: a counter that's safe to increment from two offline
wallets and reconcile later is exactly this shape. Install:
`larzscript pkg install crdt`.

```
import "crdt" as crdt
let a = crdt.gcounter_new()
let b = crdt.gcounter_new()
crdt.gcounter_inc(a, "replica-a", 3)
crdt.gcounter_inc(b, "replica-b", 5)
let merged = crdt.gcounter_merge(a, b)
print(crdt.gcounter_value(merged))    # 8 - both replicas' work survives
```

See it live, including an OR-Set example (a concurrent re-add correctly
beating a stale remove), at
[larzos.com/stack/larzcrdt/](https://larzos.com/stack/larzcrdt/).
