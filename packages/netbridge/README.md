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

## One script that does everything

[`examples/setup_and_test.lz`](examples/setup_and_test.lz) - fill in 4
lines at the top (your relay's address and a one-time-root SSH alias to
it), run it, done: generates the client key if it doesn't have one yet,
configures the relay (idempotent - safe to run every time), starts the
tunnel in the background, then actually **tests** it - connects through
the relay and checks for a real SSH banner, prints PASS/FAIL instead of
just claiming success. Verified against a real VPS, twice in a row,
before publishing this:

```
==> client key
already have /home/elevenace/.ssh/netbridge_key
==> configuring relay (larzserver)
RELAY_SETUP_OK

==> starting tunnel
relay:2200  ->  127.0.0.1:22
==> testing connection through the relay
received: SSH-2.0-dropbear_2017.75

PASS - the tunnel is live. From the relay: ssh -p 2200 you@127.0.0.1
```

(that `SSH-2.0-dropbear_2017.75` is real too - the actual banner the
client machine's own `sshd` sent back, dropbear rather than OpenSSH on
that particular box. Yours will show whatever your client runs.)

## Setting up a real relay, by hand

The same thing as the one-shot script above, broken into its three
individual steps - useful if you want to understand or customize each
piece rather than run the all-in-one version.

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
