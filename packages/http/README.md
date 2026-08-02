# lz-http

A simple HTTP client for Larzscript, built on `curl`. Install: `larzscript pkg install http`.

```
import "http" as http
let body = http.get("https://example.com")
print(http.status("https://example.com"))   # 200
http.download("https://.../file", "out")
```

Pairs with lz-json: `json.parse(http.get(url))`.
