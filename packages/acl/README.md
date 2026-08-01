# lz-acl

Simple IP allow/deny access control lists, built on `ip.in_cidr`.
`larzscript larzpkg.lz install acl`.

```
import "acl" as acl
let list = acl.new()
acl.allow(list, "10.0.0.0/8")
acl.check(list, "10.0.0.5")
```
