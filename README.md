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
| [markdown](packages/markdown) | Markdown-to-HTML, built on regex + html |
| [set](packages/set) | a real set type - union/intersect/difference |
| [stack](packages/stack) | a LIFO stack |
| [queue](packages/queue) | a FIFO queue |
| [stats](packages/stats) | mean/median/mode/stddev/percentile |
| [template](packages/template) | {{var}} string templating - loops, conditionals |
| [slugify](packages/slugify) | clean URL slugs from text |
| [case](packages/case) | camelCase/PascalCase/kebab-case converters |
| [url](packages/url) | URL parse/build + query-string encode/decode |
| [semver](packages/semver) | semantic version parse/compare/satisfies |
| [mime](packages/mime) | MIME type lookup by file extension |
| [duration](packages/duration) | parse "1d2h3m4s"-style durations to/from seconds |
| [luhn](packages/luhn) | Luhn checksum validation + check-digit generation |

### Batches 88-99 (Larz Stack migration, 2026-08)

84 more packages, ported from the old Python-based Larz Stack to real
Larzscript, one per Larz Stack library slot (`larzcart` -> `cart`, etc.):

| name | description |
|---|---|
| [actor](packages/actor) | mailbox actors |
| [agent](packages/agent) | a defensive tool-calling loop against any OpenAI-compatible chat-completions API |
| [algo](packages/algo) | classic sorting/searching algorithms that record every comparison and swap, so you can see how many steps each one actually takes on the same input |
| [archive](packages/archive) | create and list real ZIP and TAR (USTAR) archives, and extract them safely |
| [ascii](packages/ascii) | data as plain ASCII pictures |
| [barcode](packages/barcode) | EAN-13 (real check-digit math and the standard L/G/R module encodings  |
| [base](packages/base) | convert numbers between any base 2-36, with an explain() that shows the place-value math |
| [bench](packages/bench) | micro-benchmarking |
| [bits](packages/bits) | bitwise operations made visible |
| [bus](packages/bus) | an in-process message bus |
| [cache](packages/cache) | an LRU cache with optional per-entry TTL, an on-disk tier you can save()/load() for durability across runs, and memoize() to wrap a single-argument function with a cache automatically |
| [calc](packages/calc) | a real tiny expression interpreter for arithmetic strings |
| [cart](packages/cart) | shopping-cart totals with the correct order of operations |
| [cbor](packages/cbor) | RFC 8949 CBOR encode/decode, canonical (deterministic) form |
| [chart](packages/chart) | inline SVG charts |
| [command](packages/command) | the trick behind every undo button |
| [conf](packages/conf) | layered app config |
| [cron](packages/cron) | a real 5-field cron parser (minute hour day-of-month month day-of-week, plus */N step syntax) with next()/previous() run-time math, interval jobs, and a simple in-process scheduler you drive by calling tick() in your own loop |
| [currency](packages/currency) | locale-flavored money formatting/parsing on integer minor units (cents) |
| [date](packages/date) | calendar-math primitives with the algorithms in plain view |
| [db](packages/db) | a log-structured, one-file JSON database |
| [discount](packages/discount) | apply a sequence of discounts honestly |
| [email](packages/email) | build a valid MIME message from one object (text and/or HTML body plus attachments -> multipart/alternative or multipart/mixed as needed), parse() a raw message back into that same shape, and send() over real SMTP with an injectable transport for tests |
| [env](packages/env) | parse/load  |
| [events](packages/events) | a plain event emitter |
| [fake](packages/fake) | fake data for tests, demos, and DB seeding |
| [flags](packages/flags) | feature flags |
| [fsm](packages/fsm) | a declarative finite state machine |
| [fuzzy](packages/fuzzy) | approximate string matching |
| [game](packages/game) | a text-adventure engine |
| [geo](packages/geo) | geographic math |
| [graph](packages/graph) | the standard graph algorithms over an adjacency-list graph |
| [heap](packages/heap) | a readable, array-backed binary heap |
| [ical](packages/ical) | RFC 5545 iCalendar |
| [id](packages/id) | identities and verifiable credentials, sign & verify, tamper detection |
| [kata](packages/kata) | coding-exercise autograding |
| [ledger](packages/ledger) | double-entry bookkeeping |
| [limit](packages/limit) | rate limiting |
| [mark](packages/mark) | a tiny static site generator on top of `markdown` |
| [matrix](packages/matrix) | dense matrix math |
| [metrics](packages/metrics) | counters, gauges, and histograms with Prometheus text exposition format output, so any real Prometheus-compatible scraper can consume them straight from a `/metrics` endpoint |
| [migrate](packages/migrate) | a schema-migration runner |
| [netting](packages/netting) | settle a tangle of mutual debts with the fewest payments |
| [num](packages/num) | human-friendly number formatting |
| [pack](packages/pack) | MessagePack-compatible encode/decode, canonical (deterministic |
| [paginate](packages/paginate) | slice any list into pages, with the has_next/has_prev/ 1-based-index bookkeeping every list-showing page in every app needs |
| [pdf](packages/pdf) | generate a real, valid PDF |
| [phone](packages/phone) | phone number parsing |
| [pipe](packages/pipe) | composable map/filter/reduce over a list |
| [progress](packages/progress) | progress bars and a spinner for long-running CLI work |
| [prompt](packages/prompt) | interactive CLI prompts |
| [qr](packages/qr) | real QR code generation |
| [query](packages/query) | a chainable query builder over any Larzscript list of dicts |
| [quiz](packages/quiz) | self-grading quizzes |
| [retry](packages/retry) | retry a flaky call with exponential backoff, or wrap it in a circuit breaker that fails fast once it's seen enough trouble |
| [roman](packages/roman) | Roman numeral conversion (1-3999), with strict round-trip validation (rejects non-canonical forms like "IIII") and an explain() that shows the symbol-by-symbol breakdown |
| [router](packages/router) | URL routing |
| [rpc](packages/rpc) | JSON-RPC 2 |
| [sandbox](packages/sandbox) | run untrusted Larzscript code and capture its output safely |
| [search](packages/search) | a small in-process full-text search engine |
| [session](packages/session) | signed cookie sessions |
| [split](packages/split) | divide a bill among people so it reconciles exactly to the cent, using the largest-remainder method |
| [sql](packages/sql) | a parameterized SQL query builder |
| [sse](packages/sse) | Server-Sent Events |
| [state](packages/state) | checkpointed, crash-resumable workflows (sagas) |
| [store](packages/store) | a tiny document store |
| [str](packages/str) | word-game string helpers |
| [struct](packages/struct) | a few essential data structures not big enough to deserve their own package |
| [subscription](packages/subscription) | subscription billing as an explicit, testable state machine |
| [task](packages/task) | an in-process job queue |
| [text](packages/text) | small text-shaping helpers for real writing |
| [toml](packages/toml) | read AND write a real TOML subset |
| [trace](packages/trace) | execution tracing for teaching/debugging |
| [tree](packages/tree) | a readable, node-based binary search tree |
| [trie](packages/trie) | the structure behind autocomplete |
| [tui](packages/tui) | terminal-style text UI widgets |
| [turtle](packages/turtle) | classic turtle graphics (forward/turn/pen up-down), rendered headless to SVG or an ASCII grid - no display needed anywhere, including the kernel |
| [type](packages/type) | typing-test scoring |
| [unit](packages/unit) | convert values across length/mass/time/data units via a shared base-unit factor per kind, plus temperature which gets its own honest (non-linear) formulas instead of being forced into the same table |
| [vault](packages/vault) | password-based authenticated encryption with atomic file saves |
| [vm](packages/vm) | a small deterministic stack machine |
| [watch](packages/watch) | poll a directory for created/modified/deleted files matching a glob pattern |
| [ws](packages/ws) | RFC 6455 WebSocket frame encoding/decoding |
| [xml](packages/xml) | a small, safe XML |

## Adding a package

Add a new `packages/<name>/main.lz` (+ `README.md`) in a PR, then add a line
to [`larzscript`'s `packages/registry.txt`](https://github.com/larz-scripter/larzscript/blob/main/packages/registry.txt)
pointing `name` at `packages/<name>`.

## History

Each of these used to be its own repo (`lz-mathx`, `lz-json`, ...) — they were
consolidated here in 2026-07 to make the library easier to find and browse.
The old repos are archived, not deleted, so their history is intact.
