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
| [crypto](packages/crypto) | real SHA-256 + HMAC-SHA256, pure Larzscript |
| [webhook](packages/webhook) | sign/verify webhook payloads (HMAC-SHA256) |
| [apikey](packages/apikey) | generate/hash/verify opaque API keys |
| [ratelimit](packages/ratelimit) | token-bucket rate limiting |
| [invoice](packages/invoice) | build/format invoices from Money line items |
| [escrow](packages/escrow) | hold/release/refund on top of wallet/pay |
| [fx](packages/fx) | currency conversion with a settable rate table |
| [loan](packages/loan) | interest + amortization calculations |
| [markov](packages/markov) | Markov-chain text generation |
| [sentiment](packages/sentiment) | lexicon-based sentiment scoring |
| [ai](packages/ai) | pluggable LLM client, wallet-gated metered calls |
| [net](packages/net) | LarzOS's own real networking (kernel-only) |
| [ip](packages/ip) | IPv4 parse/validate/CIDR matching |
| [acl](packages/acl) | IP allow/deny access control lists |
| [validate](packages/validate) | email/URL/path-traversal/SQLi/XSS checks |
| [password_strength](packages/password_strength) | password strength scoring |
| [totp](packages/totp) | RFC 6238 one-time codes (HMAC-SHA256) |
| [jwt](packages/jwt) | signed JSON tokens (HS256) |
| [uuid](packages/uuid) | RFC 4122 v4 UUID generation |
| [yaml](packages/yaml) | lightweight YAML-subset parse/stringify |
| [diff](packages/diff) | line-based diff between two strings |
| [budget](packages/budget) | category-based budget tracking |
| [tax](packages/tax) | flat and progressive-bracket tax calculators |
| [fetch](packages/fetch) | real HTTP GET from the shell (kernel-only) |
| [regex](packages/regex) | pattern matching - literals, classes, quantifiers, anchors |

## Adding a package

Add a new `packages/<name>/main.lz` (+ `README.md`) in a PR, then add a line
to [`larzscript`'s `packages/registry.txt`](https://github.com/larz-scripter/larzscript/blob/main/packages/registry.txt)
pointing `name` at `packages/<name>`.

## History

Each of these used to be its own repo (`lz-mathx`, `lz-json`, ...) — they were
consolidated here in 2026-07 to make the library easier to find and browse.
The old repos are archived, not deleted, so their history is intact.
