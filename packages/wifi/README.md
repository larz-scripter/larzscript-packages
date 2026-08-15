# lz-wifi

Create a WiFi hotspot (access point) or join a network, using the host OS's
own already-working WiFi stack — `nmcli`/NetworkManager on Linux, `netsh`
on Windows, `networksetup` on macOS. Not a from-scratch 802.11/WPA
reimplementation, the same reasoning `ssh` binds libssh instead of
hand-rolling SSH: real WiFi security needs the OS's own drivers and crypto
stack, which are already correct and already installed.

⚠ **Hosted only** (Linux/macOS/Windows). LarzOS's bare-metal kernel has no
WiFi hardware driver at all yet — only the wired RTL8139 driver (`net`
package). This package is for Larzscript running on top of a real OS.

⚠ **Platform coverage**: Linux is the most complete (nmcli covers scan,
connect, disconnect, status, and hotspot start/stop/status). Windows covers
the same via `netsh` (hotspot needs "Virtual WiFi" driver support — check
`netsh wlan show drivers`). macOS covers client `connect()`/`status()` via
`networksetup`; hotspot creation (Internet Sharing) has no clean CLI on
macOS at all, so `hotspot_start()`/`hotspot_stop()`/`hotspot_status()`
throw there with a pointer to System Settings.

Hotspot creation generally needs root/Administrator.

`larzscript pkg install wifi`

```
import "wifi" as wifi

wifi.hotspot_start("MyHotspot", "supersecret123")
print(wifi.hotspot_status())
wifi.hotspot_stop()

for net in wifi.scan() { print(net["ssid"], net["signal"]) }
wifi.connect("HomeWifi", "wifipassword")
print(wifi.status())
wifi.disconnect()
```

Every function throws a catchable `WifiError` with a clear message on
failure (missing tool, no WiFi hardware, bad SSID/password, permission
denied).

## Testing status

Logic and platform-detection paths were tested against the real
interpreter, including the exact string-parsing for `nmcli`'s terse output
and `netsh`'s multi-line output, using realistic sample data. The
tool-missing/no-hardware failure path was verified live (a real machine
with no `nmcli` installed throws the expected `WifiError`).

`scan()`, `status()`, and `hotspot_start()`'s failure path were also
verified live on a real Windows 10 machine with real WiFi hardware. This
run caught and fixed two real bugs: `status()` was matching "connected" as
a substring of "disconnected" and always reporting `connected: true`; and
`hotspot_start()` on Windows was trusting `netsh`'s process exit code,
which returns 0 even when the hosted network genuinely fails to start —
it now parses the command's own text output instead.

**Not yet verified**: an actual hotspot broadcasting and a real device
joining it. The real hardware used above doesn't support the legacy
"hosted network" mechanism at all ("Hosted network supported: No" per
`netsh wlan show drivers`), so `hotspot_start()`'s happy path — and the
Linux/macOS hotspot paths generally — remain unverified on real WiFi
hardware. If you hit something that doesn't work as documented, please
open an issue with your OS, WiFi adapter, and the exact error.
