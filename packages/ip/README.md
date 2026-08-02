# lz-ip

IPv4 address parsing, validation, and CIDR matching, pure Larzscript.
`larzscript pkg install ip`.

```
import "ip" as ip
ip.in_cidr("192.168.1.5", "192.168.1.0/24")
ip.is_private("10.0.0.5")
```
