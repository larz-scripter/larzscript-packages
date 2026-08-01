# lz-slugify

Turn text into a clean URL slug. Install:

```
larzscript larzpkg.lz install slugify
```

```
import "slugify" as slugify
print(slugify.slug("What Is Larzscript? A Guide"))   # "what-is-larzscript-a-guide"
print(slugify.slug("Hello, World!", 5))               # "hello"
```
