# lz-ai

A generic, pluggable LLM HTTP client - point it at your own gateway or any
OpenAI-compatible endpoint via env vars (`LARZSCRIPT_AI_URL`/`_KEY`/`_MODEL`).
`ask_paid()` charges a wallet before calling out, so an AI call is a real
metered financial transaction, not just an API wrapper.
`larzscript pkg install ai`.

```
import "ai" as ai
print(ai.ask("hello"))
print(ai.ask_paid("hello", user_wallet, revenue_wallet, price_per_call))
```
