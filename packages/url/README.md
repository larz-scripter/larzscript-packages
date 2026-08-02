# lz-url

URL parsing/building and query-string encode/decode. Pure Larzscript, no
`http`/curl dependency - safe anywhere, including the kernel. Install:

```
larzscript pkg install url
```

```
import "url" as url
let u = url.parse("https://example.com/path?a=1&b=two words")
print(u["scheme"], u["host"], u["path"], u["query"]["b"])
print(url.encode("two words"))                # "two%20words"
print(url.decode("two%20words"))               # "two words"
print(url.build_query({"a": 1, "b": "x y"}))   # "a=1&b=x%20y"
```
