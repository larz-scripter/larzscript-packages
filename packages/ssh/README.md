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

The Linux x86_64 build also isn't fully static like every other platform's
release binary - Ubuntu's `libssh-dev` only ships a shared library, so this
platform's binary currently depends on libssh (and its crypto backend)
being installed on the host. Revisiting this (a real static libssh) is
part of the Windows work, since a single self-contained binary matters
most there.

## Not yet implemented

- **Server role** (`ssh.listen()`/accepting inbound connections) - libssh
  supports this, not wired up yet.
- **Remote port forwarding** (the actual `ssh -R` equivalent, what would
  replace `netbridge`'s reverse-tunnel use case) - libssh's channel API
  isn't a POSIX file descriptor, so it needs its own bridging loop rather
  than reusing this project's existing `socket_poll()` directly. Real,
  scoped follow-up work, not forgotten.
- **Host key verification** - connections aren't checked against a known-
  hosts file yet. Stated plainly: this is a real security gap for this
  phase, not silently glossed over.

## Verified

`ssh.run()` tested for real in CI against a live local `sshd` (real
`openssh-server`, key-based auth, a throwaway keypair) - connect,
authenticate, and `echo hello-from-real-sshd && whoami`, asserted against
the actual returned `stdout` and `exit_status` (both correct), not just a
clean process exit. See `.github/workflows/native.yml`'s "Verify ssh.run()
against a real local sshd" step in the `larzscript` repo for the exact
test that ran.
