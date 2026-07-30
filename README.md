# larzscript-packages

The Larzscript package library — one repo, `packages/<name>/main.lz` per
package. Install any of them with the package manager built into
[larzscript](https://github.com/larz-scripter/larzscript):

```
larzscript larzpkg.lz install mathx
```

## Packages

| name | description |
|---|---|
| [mathx](packages/mathx) | small math helpers |
| [greet](packages/greet) | friendly greetings |
| [json](packages/json) | JSON parse/stringify, pure Larzscript |
| [http](packages/http) | curl-based HTTP get/post/status/download |
| [test](packages/test) | assert/assert_eq/report |
| [color](packages/color) | ANSI terminal color |
| [csv](packages/csv) | RFC-ish CSV parse/write |
| [args](packages/args) | CLI flag parsing |
| [time](packages/time) | humanize/stopwatch/format |
| [string](packages/string) | center/wrap/reverse/snake_case/is_digit |
| [random](packages/random) | seedable PRNG (Park-Miller LCG) |
| [fs](packages/fs) | read/write/ls/copy/basename/dirname |
| [base64](packages/base64) | base64 encode/decode |
| [table](packages/table) | aligned ASCII tables |
| [log](packages/log) | leveled logging with timestamps |
| [cli](packages/cli) | build CLI tools with subcommands |
| [html](packages/html) | build HTML with auto-escaping |

## Adding a package

Add a new `packages/<name>/main.lz` (+ `README.md`) in a PR, then add a line
to [`larzscript`'s `packages/registry.txt`](https://github.com/larz-scripter/larzscript/blob/main/packages/registry.txt)
pointing `name` at `packages/<name>`.

## History

Each of these used to be its own repo (`lz-mathx`, `lz-json`, ...) — they were
consolidated here in 2026-07 to make the library easier to find and browse.
The old repos are archived, not deleted, so their history is intact.
