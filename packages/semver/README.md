# lz-semver

Semantic version (major.minor.patch) parse and compare. No pre-release/
build-metadata suffixes - a deliberately small subset. Install:

```
larzscript pkg install semver
```

```
import "semver" as semver
print(semver.compare("1.2.0", "1.10.0"))     # -1
print(semver.gt("2.0.0", "1.9.9"))           # true
print(semver.satisfies("1.5.0", "^1.2.0"))   # true (same major, >= given)
print(semver.satisfies("1.5.0", "~1.2.0"))   # false (same major.minor required)
```
