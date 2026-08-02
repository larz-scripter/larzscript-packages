# lz-template

Simple `{{var}}` string templating (Mustache-lite). Install:

```
larzscript pkg install template
```

```
import "template" as template
let t = "Hi {{name}}!\n{{#each items}}- {{this}}\n{{/each}}{{#if vip}}VIP!\n{{/if}}"
print(template.render(t, {"name": "Sam", "items": ["a", "b"], "vip": true}))
```

Supports variable substitution, `{{#each field}}...{{/each}}` loops (bind
each item as `{{this}}`, plus its own keys if it's a dict), and
`{{#if field}}...{{/if}}` conditionals. No nesting (an `#each` inside an
`#each`, or `#if` inside `#each`, isn't supported) and no dotted paths -
a deliberately small, single-pass subset.
