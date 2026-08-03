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

## One script, one password, done

[`examples/password_setup.lz`](examples/password_setup.lz) - the
fastest path if you just have a fresh relay and its root password. Fill
in 4 lines at the top (host, port, root password), run it: generates its
own client key, uses the password **once** to create a locked-down
`tunnel` user on the relay (no shell, no X11/agent forwarding - real
setup, not a stub), then connects and stays connected with real
auto-reconnect (`netbridge.supervise()`, not a one-shot attempt that
dies on the first hiccup). The password is never needed again after
that one run - every reconnect after uses the key it generated.

```
larzscript pkg install netbridge
nohup larzscript password_setup.lz > ~/netbridge.log 2>&1 &
```

Needs [`sshpass`](https://linux.die.net/man/1/sshpass) for the one-time
password step (`apt install sshpass`) - the only extra dependency, and
only for that one line.

Verified from a completely clean slate against a real VPS - no key, no
`tunnel` user, nothing - three separate times before publishing this,
each one ending with a real SSH banner confirmed back through the
tunnel (not a stub, and not the same banner every time - dropbear,
because that's what the real machine tunneled to was actually running):

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

### Zero external processes at all (no `ssh`, no `sshpass`)

[`examples/native_ssh_setup.lz`](examples/native_ssh_setup.lz) - the same
flow again, but using the [`ssh`](https://github.com/larz-scripter/larzscript-packages/tree/master/packages/ssh)
package instead of shelling out: `ssh.connect()`/`ssh.run()` for the
one-time password-authenticated relay setup, `ssh.forward_remote_port()`
(real SSH channel forwarding via libssh, the actual `ssh -R` under the
hood) for the tunnel itself. Nothing external runs at any point - not
even the `ssh` binary. Needs the `ssh` package's native libssh binding,
which is **Linux x86_64 only as of this writing** (every other platform
throws a clear `SshError` - check `ssh`'s own README for current status).
Not yet end-to-end verified against a live relay the way the script above
was (its underlying `ssh.run()`/`ssh.forward_remote_port()` calls have
been, in CI - see the `ssh` package - but this specific script's own
orchestration hasn't had its own dedicated run yet).

### On Windows

[`examples/password_setup_windows.lz`](examples/password_setup_windows.lz) -
same script, same 4 fields, but targets Windows' own built-in OpenSSH
Server instead of a Linux `sshd`. Installs/starts the Windows OpenSSH
Server feature if it isn't already running (needs one Administrator run
for that), then generates a key and connects exactly like the Linux
version. For the one-time password step it borrows WSL's `sshpass` if
WSL is installed on the machine (same trusted setup script, just reached
through WSL as a transport - `sshd` and the tunnel itself stay 100%
native Windows); without WSL it prints the setup script for you to run
by hand, once. Point of a native-Windows target at all: WSL's `sshd`
sits behind its own virtual network adapter, which is the actual cause
if a tunnel connects but times out on the far side - native Windows
doesn't have that problem.

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
