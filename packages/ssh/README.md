# lz-ssh

Real SSH - via [libssh](https://www.libssh.org/) bound into the native
`larzscript` interpreter, not a from-scratch reimplementation. Talks to
real `sshd`/OpenSSH/dropbear directly; no shelling out to the `ssh` binary.
Install: `larzscript pkg install ssh`.

```
import "ssh" as ssh

let session = ssh.connect("myserver.com", 22)
ssh.authenticate_with_password(session, "root", "hunter2")
# or: ssh.authenticate_with_key(session, "root", "~/.ssh/id_ed25519")

let result = ssh.run(session, "whoami")
print(result["stdout"])       # real command output
print(result["exit_status"])  # real exit code

ssh.disconnect(session)
```

The actual `ssh -R` equivalent - what would replace `netbridge`'s
reverse-tunnel use case, exposing a service through a relay with no inbound
port needed on the exposed machine:

```
import "ssh" as ssh

let session = ssh.connect("your-relay.example.com", 22)
ssh.authenticate_with_key(session, "you", "~/.ssh/id_ed25519")

# anyone connecting to the relay's port 2200 gets bridged to this
# machine's own sshd - blocks forever, real auto-serving
ssh.forward_remote_port(session, 2200, "127.0.0.1", 22)
```

## Why libssh, not a pure-Larzscript implementation

Real SSH key exchange needs big-integer/elliptic-curve math. Larzscript's
only numeric type is a double - `crypto`'s own README already says X25519/
Ed25519 "can't honestly be" pure Larzscript for exactly this reason. Rather
than fake it or skip real SSH entirely, this binds a real, audited C
library into the interpreter itself - the same relationship paramiko has
with OpenSSL (via Python's `cryptography` package): the actual cryptography
lives in a real library, not hand-rolled.

If you specifically want zero external C dependencies at all (nothing but
`larzscript` and this Stack), see `psktunnel` instead - real encryption,
pure Larzscript, but not SSH and meaningfully slower (measured ~1.6s per
message on this interpreter's hand-rolled SHA256 vs. libssh's real,
hardware-adjacent crypto here).

## Platform status - read this before relying on it anywhere

**Phase 1, Linux x86_64 only, client-side only.** Every function throws a
plain, catchable `SshError` ("real SSH is not available in this build") on
every other platform right now - macOS, Windows, Linux aarch64, and the
web/wasm build (the last one permanently, by the same sandbox-has-no-raw-
TCP reasoning as the `tcp`/`socket_*` builtins). Being brought up platform
by platform, hardest first-to-solve is Windows (no libssh package for the
mingw cross-compile toolchain this project's Windows build uses - needs
building libssh from source in CI, not yet done).

The Linux x86_64 release binary IS fully static (verified via `ldd` in CI -
"not a dynamic executable") - Ubuntu's `libssh-dev` ships a real static
`libssh.a` alongside its shared library, so no runtime dependency on
libssh, OpenSSL, or anything else is introduced. (It doesn't statically
link Kerberos/GSSAPI - Ubuntu ships no static Kerberos library at all - a
small stub instead provides those referenced-but-unused symbols with real
RFC 2744 "facility unavailable" responses, since this project never
authenticates via GSSAPI. See `native/gssapi_stub.c` and
`THIRD_PARTY_LICENSES.md` in the `larzscript` repo.)

`forward_remote_port()` currently supports one active forwarded connection
at a time (matches the `tcp` package's own "not concurrent" precedent) -
a second incoming connection waits until the first closes.

## Not yet implemented

- **Server role** (`ssh.listen()`/accepting inbound connections) - libssh
  supports this, not wired up yet.
- **Host key verification** - connections aren't checked against a known-
  hosts file yet. Stated plainly: this is a real security gap for this
  phase, not silently glossed over.
- **Concurrent forwarded connections** - `forward_remote_port()` is one at
  a time for now, see above.

## Verified

Both tested for real in CI against a live local `sshd` (real
`openssh-server`, key-based auth, a throwaway keypair), not stubs:

- `ssh.run()`: `echo hello-from-real-sshd && whoami`, asserted against the
  actual returned `stdout` and `exit_status` (both correct).
- `ssh.forward_remote_port()`: a real local TCP echo service, forwarded
  through the same sshd's remote port, reached by an external client - the
  real banner and a real echoed payload both round-tripped correctly.

See `.github/workflows/native.yml`'s two "Verify ssh..." steps in the
`larzscript` repo for the exact tests that ran.
