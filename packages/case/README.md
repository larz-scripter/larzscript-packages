# lz-case

Naming-convention converters (camelCase, PascalCase, kebab-case) - pairs
with `string`'s `snake()`. Install:

```
larzscript larzpkg.lz install case
```

```
import "case" as case
print(case.camel("hello world"))    # "helloWorld"
print(case.pascal("hello-world"))   # "HelloWorld"
print(case.kebab("Hello World"))    # "hello-world"
```
