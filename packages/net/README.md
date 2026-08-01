# lz-net

LarzOS's own real networking (RTL8139 + ARP/IPv4/ICMP/DNS/HTTP), wrapped
from the kernel's `/net/` virtual files. ⚠ KERNEL-ONLY - native/hosted
Larzscript has no `/net/` VFS; use the `http` package there instead.
`larzscript larzpkg.lz install net`.

```
import "net" as net
net.status()
net.ping("10.0.2.2")
net.resolve("example.com")
```
