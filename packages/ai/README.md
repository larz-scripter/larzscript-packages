# lz-ai

Call any OpenAI-compatible chat completion endpoint, in pure Larzscript. Install: `larzscript pkg install ai`.

No endpoint, model or key is ever hardcoded — configure via environment variables so this works with any real OpenAI-compatible backend (the real OpenAI API, Azure OpenAI, a self-hosted gateway, local Ollama via an OpenAI-compatible shim, ...), not one person's private infrastructure:

```
export LARZSCRIPT_AI_ENDPOINT="https://api.openai.com/v1/chat/completions"
export LARZSCRIPT_AI_MODEL="gpt-4o-mini"
export LARZSCRIPT_AI_KEY="sk-..."           # optional — omit for an endpoint that needs none
```

```
import "ai" as ai
let answer = ai.ask("What is 2 + 2? Answer with just the number.")
print(answer)
```

Skip the environment entirely with `ask_with(prompt, endpoint, model, key)` — useful for a script that talks to more than one endpoint, or shouldn't read env at all:

```
let answer = ai.ask_with("hello", "https://api.openai.com/v1/chat/completions", "gpt-4o-mini", my_key)
```

Errors raise `AiError` with a clear, specific message — missing config, no response, the endpoint's own real error message (surfaced from the response body even on a non-200 status, not swallowed), or a response shape `ask()` doesn't recognize:

```
try {
  ai.ask("hi")
} catch e {
  print(e["type"], e["message"])   # AiError  no endpoint configured - set LARZSCRIPT_AI_ENDPOINT ...
}
```
