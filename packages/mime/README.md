# lz-mime

Look up a MIME type from a filename/extension. A curated common subset
(~40 types), not the full IANA registry. Install:

```
larzscript larzpkg.lz install mime
```

```
import "mime" as mime
print(mime.type_of("photo.jpg"))   # "image/jpeg"
print(mime.type_of("weird.xyz"))   # "application/octet-stream" (fallback)
print(mime.is_text("style.css"))   # true
```
