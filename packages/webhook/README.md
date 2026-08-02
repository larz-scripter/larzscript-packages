# lz-webhook

Sign and verify webhook payloads with HMAC-SHA256 (via `crypto`).
`larzscript pkg install webhook`.

```
import "webhook" as webhook
let sig = webhook.sign(body, secret)
webhook.verify(body, received_sig, secret)
```
