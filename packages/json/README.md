# lz-json

JSON parse and stringify, in pure Larzscript. Install: `larzscript pkg install json`.

```
import "json" as json
let data = json.parse("{\"a\": [1, 2, 3], \"ok\": true}")
print(data["a"][1])                  # 2
print(json.stringify({"x": [1, "two", nil]}))   # {"x":[1,"two",null]}
```
