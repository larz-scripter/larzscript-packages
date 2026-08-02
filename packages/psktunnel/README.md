# lz-psktunnel

Remote exec + TCP forwarding through a relay, in pure Larzscript on both
ends. No `ssh`/`sshd`/`sshpass`, nothing to install beyond `larzscript`
itself and this package - not even on Windows. Install:
`larzscript pkg install psktunnel` (also needs `crypto`, `json`, `tcp`).

**This is not SSH.** It doesn't speak RFC 4253 and won't interoperate with
real `sshd`/OpenSSH/dropbear. Real SSH key exchange needs elliptic-curve/
big-integer math that Larzscript's `crypto` package can't honestly do (its
own README says as much about X25519/Ed25519 - only numeric type is a
double). This is a different, honestly-named protocol built entirely on
primitives Larzscript actually has - HMAC-SHA256 AEAD under a key derived
from a password both sides already know - solving the same practical
problem (remote exec + forwarding through a relay) a different way.

```
import "psktunnel" as psk

# on the relay (a public server, stays up):
let key = psk.derive_key("a shared password")
let cfg = psk.default_config()
cfg["master_key"] = key
cfg["forward_public_port"] = 2200   # 0 = exec only, no forwarding
psk.relay_daemon(cfg)

# on the machine you want to expose (dials OUT, works behind NAT/CGNAT):
let cfg2 = psk.default_config()
cfg2["relay_host"] = "your.relay.example.com"
cfg2["master_key"] = key
cfg2["forward_public_port"] = 2200
cfg2["target_port"] = 22
psk.run_client(cfg2)
```

## Read this before you use it: real, measured performance

This interpreter's SHA256 is hand-rolled Larzscript (bit-by-bit rotate/and/
xor), not a C/hardware implementation - it's genuinely slow, and this
package's honesty is in stating the real numbers rather than a
back-of-envelope guess:

- **Per-connection setup** (`_new_chan`, 4 HMAC calls to derive send/recv
  keys once): **~6 seconds**. Paid once per control connection and once per
  forwarded connection - not per message.
- **Per-message send or receive**: **~1.6 seconds each way** (measured, not
  estimated - see the package's git history for the actual benchmark run).
  `exec()` needs two full round trips (request, response) on top of the
  command's own execution time - **budget 15-20+ seconds for one `exec()`
  call.**
- **`forward()`**: correctness proven end-to-end (a real local TCP service's
  banner and an echoed payload both round-tripped through two separate
  processes and a real relay), but at ~1.6s of latency per chunk each way -
  **not usable for anything interactive** (a shell session through this
  would feel like multiple seconds per keystroke). Fine for occasional,
  non-interactive request/response traffic; not a substitute for a live
  terminal.

If you need interactive speed, this isn't it - use real `ssh` (see
`netbridge` in this same Stack) or, if you specifically need SSH built into
Larzscript itself rather than shelling out, that needs binding a real C SSH
library into the native interpreter (a different, much bigger project - see
`larzscript`'s own issue tracker).

## A real, known correctness limit: binary data with a `0x00` byte

Larzscript's strings are NUL-terminated C strings under the hood -
`len()`/`for`-in/concatenation all resolve to `strlen()` (confirmed by
reading `native/larzscript.c`'s `bi_len`), so a raw byte value of 0 anywhere
in a "string" value silently truncates everything after it. This package's
own framing (sequence numbers, hex-encoded ciphertext, JSON control
messages) is entirely ASCII/hex text and is unaffected. But `forward()`'s
actual payload - raw bytes read from whatever local TCP service you're
exposing - is not: if that traffic ever contains a real `0x00` byte, it
will be silently truncated on the way through. Verified safe for
text-oriented protocols (HTTP, line-based protocols); **not verified safe
for arbitrary binary protocols** that may contain NUL bytes.

## Security model, stated plainly

- Authentication is "does your `master_key` (derived from the shared
  password) successfully decrypt the handshake" - a wrong password fails to
  authenticate rather than silently connecting.
- Key derivation is `crypto.pbkdf2` (HMAC-SHA256-based), iterations
  deliberately modest given the per-iteration cost on this interpreter -
  weaker than a production KDF like Argon2/bcrypt against an attacker who
  captures the derivation and brute-forces it offline.
- No forward secrecy: one long-term pre-shared key for the connection's
  whole life, not an ephemeral per-session key exchange (that would need
  real Diffie-Hellman, the exact big-integer wall this design exists to
  avoid).
- Every message gets its own non-overlapping keystream region (derived from
  a fixed per-direction key plus a per-message counter offset, not a fresh
  key per message) - the standard, correct way real stream ciphers avoid
  keystream reuse, chosen specifically because re-deriving fresh HMAC keys
  per message was the dominant cost in early benchmarking (~3.8s/message
  down to ~1.6s/message after this change).

## Capabilities

- **`relay_exec(admin_host, admin_port, cmd)`** - from any short-lived
  script that can reach the relay's admin port (127.0.0.1-only by
  convention, plaintext, meant to be run *on* the relay box or tunneled to
  it), have the connected client run `cmd` and return its output.
- **`forward()`** (via `cfg["forward_public_port"]`) - one active forwarded
  TCP connection at a time (matches the `tcp` package's own "not
  concurrent" precedent). Anyone connecting to `relay:forward_public_port`
  gets bridged to `target_host:target_port` on the client machine.

Both ride the same persistent control connection the client dials out to
the relay, with real auto-reconnect + exponential backoff
(`retry_min`/`retry_max` in the config) if the relay or network hiccups.

## Verified

Two separate real processes (a `relay_daemon()` and a `run_client()`), real
sockets, real encryption end to end:
- `relay_exec()` ran a real shell command on the client process and
  returned the real output.
- `forward()` bridged a real local TCP echo service through the relay to an
  external client - a real banner and a real echoed payload both
  round-tripped correctly.
- Reconnect behavior verified: killing and restarting the target service
  the client forwards to, the client's own error path correctly closes and
  reconnects (a real bug here - the control socket wasn't being closed on
  error, leaving the relay stuck serving a dead session - was found via
  this exact test and fixed, not just fixed and assumed working).
