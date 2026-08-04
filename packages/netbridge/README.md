# lz-netbridge

Reverse SSH tunnels through a relay you control - expose a service on a
machine behind NAT/CGNAT/a firewall, with no inbound port opened on that
machine - plus direct pull/push file transfer for when you don't even
need a tunnel (see below). Install: `larzscript pkg install netbridge`
(pulls in the [`ssh`](https://github.com/larz-scripter/larzscript-packages/tree/master/packages/ssh)
package too - real SSH via libssh bound into the interpreter itself.
**No external `ssh`/`scp`/`sshpass` binary anywhere**, on any platform
`ssh` is linked on - see that package's own README for exact platform
coverage).

```
import "netbridge" as netbridge

let srv = netbridge.default_server()
srv["host"] = "relay.example.com"
srv["identity"] = "~/.ssh/netbridge_key"
let shell = netbridge.tunnel_forward("my-shell", 2200, "127.0.0.1", 22)
print(netbridge.route_label(shell))
netbridge.supervise(srv, shell, {"log_fn": fn(name, line) { print(line) }})
```

Two tunnel kinds, both real SSH channel forwarding (no VPN, no extra
daemon on the relay beyond `sshd`): `tunnel_forward()` (relay:PORT → one
service on this machine, e.g. its own `sshd` - "log into me through the
relay") or `tunnel_socks()` (a real SOCKS4/4a/5 proxy on the relay side -
one tunnel reaches your whole reachable network, not just one port).
`supervise()` connects, hands off to the `ssh` package's
`forward_remote_port()`/`forward_remote_socks()` (which block for as
long as the tunnel stays up, retrying individual bad connections
internally without dropping the whole tunnel), and reconnects with
exponential backoff the moment the underlying session actually dies
(`ssh_is_connected()` - verified live by killing a session mid-tunnel and
confirming reconnect within about a second, not just assumed).

## pull() / push() - direct file transfer, no tunnel needed at all

If this machine can already reach the relay/server directly (most
residential/office networks allow *outbound* even when they can't accept
*inbound* - that's the whole reason tunnels exist in the first place),
moving a file doesn't need a tunnel, just an authenticated connection:

```
import "netbridge" as netbridge

let srv = netbridge.default_server()
srv["host"] = "your-server.example.com"
srv["identity"] = "~/.ssh/id_ed25519"
netbridge.pull(srv, "/remote/backup.tar.gz", "backup.tar.gz")
netbridge.push(srv, "local-file.bin", "/remote/local-file.bin")
```

Each file transfers in fixed-size chunks over `ssh.run()` (the `ssh`
package has no dedicated SFTP subsystem yet), independently retried with
backoff, and skipped entirely if the destination already has a matching
size - a killed/failed transfer just resumes on re-run rather than
starting over. Chunk size differs by direction and isn't just a tuning
knob: `pull()`'s base64 comes back through the channel's normal DATA
stream (no real size limit), but `push()`'s base64 goes the other way -
embedded directly in the exec **command string** itself - which hits
real SSH/shell command-length limits. Found live: a large single chunk
there got the whole connection reset by the server, not a clean error.
Binary safety (a real concern - anything compressed/encrypted routinely
contains `0x00` bytes, which the ordinary string-based path in this
language silently drops, see the `ssh` package's own notes on this)
comes from decoding straight into a byte LIST, never reconstructed as a
string - verified with real files up to 700KB checksummed byte-exact
end to end, not just small test payloads.

## One script, one password, done - same file on Linux, macOS, or Windows

[`examples/native_ssh_setup.lz`](examples/native_ssh_setup.lz) - the one
script to reach for, on any of them. Fill in 6 lines at the top
(relay host/port, root password, the port/service to expose), run it:
generates its own client key, uses the password **once** to create a
locked-down `tunnel` user on the relay (no shell, no X11/agent
forwarding - real setup, not a stub), then connects and stays connected
with real reverse-forwarding (`ssh.forward_remote_port()`, real SSH
channel forwarding via libssh - the actual `ssh -R` under the hood). The
password is never needed again after that one run - every reconnect
after uses the key it generated.

Uses the [`ssh`](https://github.com/larz-scripter/larzscript-packages/tree/master/packages/ssh)
package (real SSH via libssh, bound into the interpreter itself) for
everything - `ssh.connect()`/`ssh.run()` for the one-time relay setup,
`ssh.forward_remote_port()` for the tunnel, `ssh.verify_host_trust_on_first_use()`
checking the relay's host key on every connection (not blind trust -
fails closed if it ever changes). Nothing external runs at any point on
any platform - no `ssh` binary, no `sshpass`, no WSL:

```
larzscript pkg install netbridge
larzscript pkg install ssh
nohup larzscript native_ssh_setup.lz > ~/netbridge.log 2>&1 &
```

(on Windows: just `larzscript native_ssh_setup.lz` - no `nohup`/`&`.)

Needs the `ssh` package's native libssh binding, which as of this
writing covers every native target larzscript builds for - Linux
(x86_64 + aarch64), macOS (x86_64 + arm64), and Windows x86_64 - all
CI-verified end to end against a real independent `sshd`/`ssh` client
(see `ssh`'s own README for exact status). On the one platform it
doesn't cover (wasm, permanently - no raw TCP in that sandbox)? Use
[`examples/password_setup.lz`](examples/password_setup.lz) instead -
same 4-field flow, shells out to the real `ssh`/`sshpass` binaries
instead, works wherever those are installed (`apt install sshpass`),
verified from a completely clean slate against a real VPS three separate
times before publishing:

```
==> client key
generated /home/you/.ssh/netbridge_key
==> setting up the relay (password, one time only)
RELAY_SETUP_OK
==> connecting (auto-reconnects if the relay or network hiccups)
relay:2200  ->  127.0.0.1:22
```

(from the relay: `ssh -p 2200 you@127.0.0.1` now lands on the client
machine's real `sshd`.)

[`examples/password_setup_windows.lz`](examples/password_setup_windows.lz)
also still exists for the one Windows-specific case `native_ssh_setup.lz`
doesn't cover: exposing Windows' own **OpenSSH Server** feature
specifically (installs/starts it if missing) rather than an arbitrary
local port - `native_ssh_setup.lz`'s `TARGET_PORT` can already point at
anything reachable on 127.0.0.1 (22 if you have Windows OpenSSH Server
running, 3389 for RDP, etc.), so reach for the Windows-specific script
only if you also want this script to install that Windows feature for
you.

### Nothing to install at all - not even a real sshd on the exposed machine

[`examples/native_ssh_server_and_bridge.lz`](examples/native_ssh_server_and_bridge.lz) -
for when the machine you're exposing has no real SSH server running (or
you just don't want to set one up). Instead of forwarding to something
that already exists on `TARGET_HOST:TARGET_PORT`, the "service" being
exposed is this machine's OWN native `ssh.listen()`/`ssh.serve()` server -
real interactive shell (`ssh -p 2200 you@127.0.0.1` lands on an actual
login shell with a real pty, not just single commands), still nothing
external anywhere. Two `larzscript` processes cooperate under one command
(this interpreter has no OS threads, so the accept loop and the tunnel
loop can't share a process) - one user-facing `nohup ... &` still starts
both. See the file's own header comment for exactly how, and how to stop
both cleanly.

Dynamic and resilient, not a one-shot attempt - found needed during real
end-to-end testing against a live relay, not designed in the abstract:

- **Real connect timeouts** (`ssh.connect(host, port, timeout_seconds)`,
  default 15s) - won't hang forever against an unreachable or
  silently-firewalled relay.
- **Auto-picks the next free port** if `TUNNEL_PORT`/`LOCAL_SERVER_PORT`
  are already taken (e.g. a previous run still holding it), instead of
  just erroring out.
- **Real worker-readiness check** - polls for the native server to
  actually be listening before connecting the tunnel to it, instead of a
  fixed blind sleep that either wastes time or races a slower machine.
- **Auto-reconnects with exponential backoff** if the relay connection
  ever drops (same backoff shape as `supervise()` above) - restarts the
  native server too if it died in the meantime.
- **Specific error messages per failure mode** - unreachable relay, bad
  password, changed host key, and relay-script failure are each reported
  distinctly instead of one generic failure.

## Setting up a real relay by hand, with your own SSH key

If you'd rather not put a root password in a script - the same setup,
authenticated with an SSH key you already have instead:
[`examples/setup_and_test.lz`](examples/setup_and_test.lz) (fill in a
`~/.ssh/config` alias with root access instead of a password) also
self-tests before reporting success. Or broken into its three individual
steps below, if you want to understand or customize each piece.

**1. On the machine you want to expose** (the "client" - wherever this
runs FROM):

```sh
mkdir -p ~/.ssh && chmod 700 ~/.ssh
ssh-keygen -t ed25519 -f ~/.ssh/netbridge_key -N "" -C "netbridge-$(hostname)"
cat ~/.ssh/netbridge_key.pub   # copy this - you'll paste it in step 2
```

**2. On the relay** (any server you control, reachable from the client -
run as root):

```sh
TUNNEL_USER="tunnel"
PUBKEY='paste the netbridge_key.pub contents from step 1 here'

id -u "$TUNNEL_USER" >/dev/null 2>&1 || useradd -m -s /usr/sbin/nologin "$TUNNEL_USER"
install -d -m 700 -o "$TUNNEL_USER" -g "$TUNNEL_USER" "/home/$TUNNEL_USER/.ssh"
touch "/home/$TUNNEL_USER/.ssh/authorized_keys"
chmod 600 "/home/$TUNNEL_USER/.ssh/authorized_keys"
chown "$TUNNEL_USER:$TUNNEL_USER" "/home/$TUNNEL_USER/.ssh/authorized_keys"
printf 'no-pty,no-X11-forwarding,no-agent-forwarding,command="echo netbridge: no interactive shell" %s\n' "$PUBKEY" \
  >> "/home/$TUNNEL_USER/.ssh/authorized_keys"
```

`tunnel` can now forward ports through the relay and do nothing else - no
shell (`command=` forces a harmless echo instead of whatever the client
asked for, `no-pty` blocks an interactive session outright), no X11/agent
forwarding. Reverse-forwarded ports stay reachable only from the relay
itself (`GatewayPorts no`, the default) unless you deliberately turn that
on in `sshd_config` - a safe default, not an oversight.

**3. Back on the client** - connect:

```
import "netbridge" as netbridge

let srv = netbridge.default_server()
srv["host"] = "your.relay.example.com"   # or its IP
srv["port"] = 22                          # whatever port sshd listens on there
srv["user"] = "tunnel"
srv["identity"] = env("HOME", ".") + "/.ssh/netbridge_key"
srv["known_hosts"] = env("HOME", ".") + "/.ssh/known_hosts"

let shell = netbridge.tunnel_forward("my-shell", 2200, "127.0.0.1", 22)
netbridge.supervise(srv, shell, {"log_fn": fn(name, line) { print("[" + name + "] " + line) }})
```

Run it (it blocks - this is the live tunnel, run it under whatever your OS
uses to keep a process alive: `systemd`, `nohup ... &`, a supervisor). From
the relay, `ssh -p 2200 you@127.0.0.1` now lands on the client machine's
real `sshd` - verified live: connecting to the relay's port 2200 returns
the client's actual SSH banner (`SSH-2.0-...`), not a stub.
