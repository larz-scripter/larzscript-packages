# lz-csv

Parse and write CSV in pure Larzscript (handles quoted fields). `larzscript larzpkg.lz install csv`.

```
import "csv" as csv
let rows = csv.parse("a,b\n1,\"x,y\"")   # [["a","b"],["1","x,y"]]
print(csv.to_csv(rows))
```
