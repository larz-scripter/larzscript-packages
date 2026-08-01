# lz-set

A real set type, built on a dict-as-membership-map. Install:

```
larzscript larzpkg.lz install set
```

```
import "set" as set
let a = set.from_list([1, 2, 3])
let b = set.from_list([2, 3, 4])
print(set.to_list(set.union(a, b)))          # [1, 2, 3, 4]
print(set.to_list(set.intersect(a, b)))       # [2, 3]
print(set.to_list(set.difference(a, b)))      # [1]
print(set.to_list(set.symmetric_difference(a, b)))  # [1, 4]
print(set.has(a, 2))                          # true
print(set.is_subset(set.from_list([2, 3]), a))  # true
```

Every function takes/returns a plain dict, so a set stays easy to inspect
or `print` - there's no opaque `Set` class to unwrap.
