# lz-yaml

A lightweight YAML-subset parser/serializer - nested mappings, simple
scalar lists, `#` comments. Not full YAML (no anchors/flow-style/multi-doc).
`larzscript larzpkg.lz install yaml`.

```
import "yaml" as yaml
let data = yaml.parse("name: Ada\ntags:\n  - dev\n  - admin\n")
print(yaml.stringify(data))
```
