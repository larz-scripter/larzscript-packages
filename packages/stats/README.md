# lz-stats

Basic descriptive statistics on a list of numbers. Complements `mathx`
(number theory) rather than overlapping it. Install:

```
larzscript pkg install stats
```

```
import "stats" as stats
let xs = [4, 8, 15, 16, 23, 42]
print(stats.mean(xs))
print(stats.median(xs))
print(stats.mode([1, 2, 2, 3]))
print(stats.variance(xs))
print(stats.stddev(xs))
print(stats.percentile(xs, 90))
print(stats.range_of(xs))
```

`stddev`/`variance` are population statistics (divide by n, not n-1).
