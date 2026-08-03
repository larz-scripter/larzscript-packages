# lz-netbridge

Reverse SSH tunnels through a relay you control - expose a service on a
machine behind NAT/CGNAT/a firewall, with no inbound port opened on that
machine. Install: `larzscript pkg install netbridge`.

```
import "netbridge" as netbridge

let srv = netbridge.default_server()
srv["host"] = "relay.example.com"
srv["identity"] = "~/.ssh/netbridge_key"
let shell = netbridge.tunnel_forward("my-shell", 2200, "127.0.0.1", 22)
print(netbridge.route_label(shell))
print(netbridge.build_ssh_command(srv, shell))
netbridge.supervise(srv, shell, {"log_fn": fn(name, line) { print(line) }})
```

Two tunnel kinds, both plain `ssh -R` (no VPN, no extra daemon on the
relay beyond `sshd`): `tunnel_forward()` (relay:PORT → one service on this
machine, e.g. its own `sshd`) or `tunnel_socks()` (a reverse dynamic/SOCKS5
forward - one tunnel reaches your whole reachable network).
`supervise()` runs a real ssh child, detects failure by scanning for known
error markers (`ssh -N` is silent on success, so there's no exit code to
wait on), and reconnects with exponential backoff.

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
`ssh.forward_remote_port()` for the tunnel. Nothing external runs at any
point on any platform - no `ssh` binary, no `sshpass`, no WSL:

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
