# lz-duration

Parse compact duration strings ("1d2h3m4s", "90s") into seconds, and back -
the reverse direction of `time`'s `humanize()`. Install:

```
larzscript larzpkg.lz install duration
```

```
import "duration" as duration
print(duration.parse("1d2h3m4s"))   # 93784
print(duration.format(93784))       # "1d2h3m4s"
```
