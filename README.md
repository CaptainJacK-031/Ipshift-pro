# IPShift Pro — Kali Linux

Change your public IP address through VPNs, with leak protection and a firewall kill switch.
One project: a root daemon (`ipshiftd`), a command line (`ipshift`) and a PySide6 window (`ipshift-gui`).

> **Read this first: what "anonymous" means here.**
> No program can promise maximum anonymity, and this one does not. It does the network part as strictly as
> it can: the kill switch is fail-closed, DNS and IPv6 are locked down, every new address is verified, and
> nothing leaves your machine outside the tunnel. A VPN still only *moves* trust to its operator, and
> changing the address does nothing about your browser, your accounts or an observer who sees both ends.
> Frequent rotation can even make things worse in one way: every reconnect shows your real address to the
> VPN server you connect to. The full list is in [What this does not do](#what-this-does-not-do).

## What you get

| | |
|---|---|
| System | Kali Linux (any Debian-based system with systemd and nftables should do) |
| Language | Python 3.10+. The daemon, engine and command line use the standard library only. |
| Window | PySide6 (optional; the command line does everything the window does) |
| VPN | Free, paid and self-hosted servers, as OpenVPN (`.ovpn`) or WireGuard (`.conf`) profiles, each tagged with its tier |
| Rotation | From every 1 second. The 1-second floor is enforced in the settings, the scheduler and the command line. |
| Monitoring | Public IP (verified by independent services) and private addresses, on Wi-Fi and mobile hotspots |
| Leak tests | DNS (configuration, resolution path, escape), IPv6, tunnel, kill switch, IPv4 escape, LAN exposure, MAC address |
| Kill switch | nftables, fail-closed, armed before any tunnel comes up |
| History | SQLite, pruned after 30 days by default |

## Install

### Kali / Debian: the package

```
sudo apt install ./ipshift-pro_1.0.0_all.deb
```

It needs Python 3.10+, nftables, iproute2 and `wireguard-tools` or `openvpn`, and recommends the window's
toolkit (`python3-pyside6.qtwidgets`). The daemon is **not** started automatically.

### From the sources

```
tar xzf ipshift-pro-1.0.0.tar.gz && cd ipshift-pro-1.0.0
sudo apt install nftables iproute2 wireguard-tools openvpn python3-pyside6.qtwidgets
sudo packaging/install.sh
```

The installer copies the program to root-owned places (the daemon runs code from there, so it refuses to
install into a place other users can write to), creates the group `ipshift`, writes the launchers, the systemd
unit, the desktop entry, the completion and the manual pages. `packaging/uninstall.sh` takes it all out again
and gives you the normal network back first. `--prefix`, `--add-user NAME` and `--enable` are explained by
`--help`.

### First steps

```
sudo usermod -aG ipshift $USER          # then log out and in again
sudo systemctl enable --now ipshiftd    # start the daemon, now and at every boot
ipshift doctor                          # is this machine ready?
```

**Membership of the group `ipshift` is the right to control this machine's network** (firewall, routes,
tunnels, DNS). Add only people you would trust with that.

### Try it without root

Nothing real is touched in simulation mode: a pretend network with demo VPN profiles.

```
ipshiftd --simulate &          # as a normal user
ipshift status
ipshift-gui --demo             # the window against its own private simulated daemon
```

## Use it

```
ipshift vpn import ~/vpn/                 # every .ovpn / .conf file in a folder (asks where each server comes from)
ipshift vpn connect                       # connect, and verify the new public IP
ipshift rotate --interval 1               # change the public IP as fast as the VPNs allow
ipshift status --watch                    # live view
ipshift privacy-check                     # DNS, IPv6 and kill switch tests
ipshift stop                              # stop rotating, disconnect, give the normal network back
```

| Command | What it does |
|---|---|
| `ipshift status` | protection level, VPN, public and private IP, rotation, alerts (`--watch`, `--check` for scripts) |
| `ipshift ip` | the public and private addresses (`--public`, `--private`; `--direct` looks up your real address, only possible with no VPN and no kill switch) |
| `ipshift vpn …` | `connect`, `disconnect`, `cancel`, `list`, `import`, `remove`, `enable`, `disable`, `auth`, `set`, `reset` |
| `ipshift rotate` | `--interval SECONDS` (minimum 1), `--now`, `--stop`, `--no-immediate`, `--no-follow`, `--stop-on-exit` |
| `ipshift stop` | normal: disconnect and release the firewall. `--keep-blocking`: disconnect but keep blocking everything. `--rotation-only`: stop rotating, stay connected. |
| `ipshift privacy-check` | the ten tests with a score and a verdict (`--quick`, `--only`, `--strict`, `-v`); exit status 4 when it finds a problem |
| `ipshift killswitch` | `status`, `on`, `off`, `lockdown` (block everything, the VPN too) |
| `ipshift config` | `show`, `get`, `set`, `reload` |
| `ipshift history` | show, `--export FILE` as CSV, `--clear` |
| `ipshift harden` | DNS / IPv6 / MAC protections: `status`, `mac enable|disable|randomize|restore` |
| `ipshift restore` | undo everything IPShift changed: firewall, DNS, IPv6, MAC addresses, stray tunnels. Works without the daemon. |
| `ipshift doctor`, `events`, `version` | readiness check, recent alerts, versions |

`ipshift --json …` gives machine-readable output. Exit codes: 0 success, 1 failed, 2 wrong usage, 3 daemon not
reachable, 4 privacy check found problems, 130 interrupted. Manual pages: `man ipshift`, `man ipshiftd`,
`man ipshift-gui`.

### The window

`ipshift-gui` has five pages: **Dashboard** (public and private IP, VPN status, the protection banner, the
interval control, Start and Stop), **Privacy** (the DNS and IPv6 leak tests and the rest), **History** (every
rotation), **Profiles** (import and manage VPNs) and **Settings**. Connection failures show up as alerts in the
window and in the system tray. Anything that lowers your protection asks for confirmation first, and your
real address stays hidden until you ask to see it. Closing the window leaves the VPN and the rotation running.
