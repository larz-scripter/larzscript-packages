# lz-mathx

Small pure-Larzscript math helpers. Install with the Larzscript package manager:

```
larzscript larzpkg.lz install mathx
```

Then in your program:

```
import "mathx" as m
print(m.mean([1, 2, 3, 4]))    # 2.5
print(m.fib(10))               # 55
print(m.primes_up_to(20))      # [2, 3, 5, 7, 11, 13, 17, 19]
```
