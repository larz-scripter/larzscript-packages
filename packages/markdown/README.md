# lz-markdown

A small Markdown-to-HTML converter, built on `regex` and `html`. Install:

```
larzscript pkg install markdown
```

```
import "markdown" as markdown
print(markdown.to_html("# Hi\n\nSome **bold** text and a [link](/x)."))
```

Supports: `#`/`##`/`###` headings, `**bold**`, `` `code` ``, `[text](href)`
links, fenced ` ``` ` code blocks, `-`/`*` bullet lists, paragraphs. No
tables, no nested lists, no blockquotes - a deliberately small, correct
subset, not a full CommonMark implementation.
