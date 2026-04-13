---
link: https://github.com/matthart1983/netwatch
site: GitHub
excerpt: Real-time network diagnostics in your terminal. One command, zero
  config, instant visibility. - matthart1983/netwatch
twitter: https://twitter.com/@github
slurped: 2026-04-11T19:02
title: "GitHub - matthart1983/netwatch: Real-time network diagnostics in your
  terminal. One command, zero config, instant visibility."
---
[wiki](https://github.com/matthart1983/netwatch/wiki)

**Real-time network diagnostics in your terminal. One command, zero config, instant visibility.**

[![crates.io](https://camo.githubusercontent.com/6786327c6c013b246b75a4bb29b719fe577b6f8b0a89b2c8d8c341c143d8e064/68747470733a2f2f696d672e736869656c64732e696f2f6372617465732f762f6e657477617463682d7475692e737667)](https://crates.io/crates/netwatch-tui) [![Release](https://camo.githubusercontent.com/4dc6a55579acf879ca2cb079366f1437b11a9b9ac745ed8a3671e9a55f92ca25/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f762f72656c656173652f6d61747468617274313938332f6e65747761746368)](https://github.com/matthart1983/netwatch/releases) [![Platform](https://camo.githubusercontent.com/a9b58fc7ebedd261b2fb621777fe22e461f1e0f930216db308feff39948eaefb/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f706c6174666f726d2d6d61634f532532302537432532304c696e75782d626c7565)](https://camo.githubusercontent.com/a9b58fc7ebedd261b2fb621777fe22e461f1e0f930216db308feff39948eaefb/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f706c6174666f726d2d6d61634f532532302537432532304c696e75782d626c7565) [![License](https://camo.githubusercontent.com/f8df3091bbe1149f398a5369b2c39e896766f9f6efba3477c63e9b4aa940ef14/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c6963656e73652d4d49542d677265656e)](https://camo.githubusercontent.com/f8df3091bbe1149f398a5369b2c39e896766f9f6efba3477c63e9b4aa940ef14/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c6963656e73652d4d49542d677265656e) [![Wiki](https://camo.githubusercontent.com/310d5488ff36a6552cd554349a5a3e8dd4ecd54b9301fe8b20fa078a3c8d4cbe/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f646f63732d57696b692d626c75653f6c6f676f3d676974687562)](https://github.com/matthart1983/netwatch/wiki)

[![NetWatch — Dashboard, Connections, Packets, Topology](https://github.com/matthart1983/netwatch/raw/main/demo.gif)](https://github.com/matthart1983/netwatch/blob/main/demo.gif)

_Launch → see every interface, connection, and health probe instantly. Arm the flight recorder before an incident disappears._

---

## Install

[](https://github.com/matthart1983/netwatch#install)

# Homebrew (macOS / Linux)
brew install matthart1983/tap/netwatch

# Cargo
cargo install netwatch-tui

# Pre-built binaries — see Releases

**All platforms & options**

| Platform              | Download                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------------- |
| Linux (x86_64)        | [`netwatch-linux-x86_64.tar.gz`](https://github.com/matthart1983/netwatch/releases/latest)  |
| Linux (aarch64)       | [`netwatch-linux-aarch64.tar.gz`](https://github.com/matthart1983/netwatch/releases/latest) |
| macOS (Intel)         | [`netwatch-macos-x86_64.tar.gz`](https://github.com/matthart1983/netwatch/releases/latest)  |
| macOS (Apple Silicon) | [`netwatch-macos-aarch64.tar.gz`](https://github.com/matthart1983/netwatch/releases/latest) |

**From source:**

git clone https://github.com/matthart1983/netwatch.git && cd netwatch
cargo build --release

**Prerequisites:** Rust 1.70+, libpcap (`sudo apt install libpcap-dev` on Linux, included on macOS)

## Quick Start

[](https://github.com/matthart1983/netwatch#quick-start)

netwatch            # Interface stats, connections, config
sudo netwatch       # Full mode — adds health probes + packet capture
netwatch --generate-config

### Flight Recorder

[](https://github.com/matthart1983/netwatch#flight-recorder)

Catch transient failures that vanish before you can inspect them:

```
Shift+R   Arm a rolling 5-minute recorder
Shift+F   Freeze the current incident window
Shift+E   Export an incident bundle to ~/netwatch_incident_YYYYMMDD_HHMMSS/
```

Each bundle includes `summary.md`, `connections.json`, `health.json`, `bandwidth.json`, `dns.json`, `alerts.json`, `manifest.json`, and `packets.pcap` when capture data is available.

---

## Why NetWatch?

[](https://github.com/matthart1983/netwatch#why-netwatch)

Most network tools make you choose: **see what's happening** (iftop, bandwhich) or **inspect packets** (Wireshark, tshark). NetWatch does both in a single terminal — from a 10,000-foot dashboard view down to individual packet bytes.

| What you get                                    | How fast     |
| ----------------------------------------------- | ------------ |
| Every interface with live RX/TX sparklines      | **Instant**  |
| Every connection with process name + PID        | **Instant**  |
| Gateway & DNS health with latency heatmap       | **Instant**  |
| Wireshark-style packet capture + decode         | One keypress |
| Rolling incident capture + frozen export bundle | One keypress |
| Network topology map with traceroute            | One keypress |
| PCAP export for offline analysis                | One keypress |

**No config files. No setup. No flags required.**

---

## Screenshots

[](https://github.com/matthart1983/netwatch#screenshots)

---

## Features

[](https://github.com/matthart1983/netwatch#features)

### 🖥️ Dashboard

[](https://github.com/matthart1983/netwatch#%EF%B8%8F-dashboard)

Everything at a glance — interfaces, aggregate bandwidth graph, top connections, gateway/DNS health probes, and a color-coded latency heatmap. Useful in 5 seconds.

### 🔌 Connections

[](https://github.com/matthart1983/netwatch#-connections)

Every open socket with **process name**, PID, protocol, state, remote address, GeoIP location, and per-connection **latency sparklines**. Sort by any column, jump to filtered packet view.

### 📡 Interfaces

[](https://github.com/matthart1983/netwatch#-interfaces)

Per-interface detail: IPv4/IPv6 addresses, MAC, MTU, total RX/TX with individual sparkline history, errors, and drops.

### 📦 Packet Capture

[](https://github.com/matthart1983/netwatch#-packet-capture)

Live capture with deep protocol decoding — **DNS** (queries, types, response codes), **TLS** (version, SNI), **HTTP** (method, path, status), **ICMP**, **ARP**, **DHCP**, **NTP**, **mDNS**, and 25+ service labels. TCP stream reassembly, handshake timing, display filters, BPF capture filters, bookmarks, and PCAP export.

### 📈 Processes

[](https://github.com/matthart1983/netwatch#-processes)

Per-process bandwidth ranking with live RX/TX rates, totals, and connection counts. Useful for spotting the process behind a noisy host or bandwidth spike.

### 🎥 Flight Recorder

[](https://github.com/matthart1983/netwatch#-flight-recorder)

Arm a rolling 5-minute capture window, then freeze it manually or when a critical network-intel alert fires. Export a self-contained incident bundle with a human-readable summary, `.pcap`, connection/process context, health samples, DNS analytics, and alert history.

### 🗺️ Topology

[](https://github.com/matthart1983/netwatch#%EF%B8%8F-topology)

ASCII network map showing your machine, gateway, DNS servers, and top remote hosts with connection counts and color-coded health indicators. Built-in **traceroute** from any host.

### 📊 Stats

[](https://github.com/matthart1983/netwatch#-stats)

Protocol hierarchy table with packet counts, byte totals, and distribution bars. TCP handshake histogram with min/avg/median/p95/max.

### ⏱️ Timeline

[](https://github.com/matthart1983/netwatch#%EF%B8%8F-timeline)

Gantt-style connection timeline — when each connection was active, color-coded by TCP state. Adjustable windows from 1 minute to 1 hour.

### ⚙️ Settings

[](https://github.com/matthart1983/netwatch#%EF%B8%8F-settings)

Built-in settings overlay for theme, default tab, refresh rate, capture interface, packet-follow mode, GeoIP paths, BPF filter, and alert thresholds. Use `,` to open it and `S` to persist changes.

---

## Display Filters

[](https://github.com/matthart1983/netwatch#display-filters)

Wireshark-style filter syntax in the Packets tab:

```
tcp                        # Protocol
192.168.1.42               # IP address (src or dst)
ip.src == 10.0.0.1         # Directional
port 443                   # Port
stream 7                   # Stream index
contains "hello"           # Text search
tcp and port 443           # Combinators
!dns                       # Negation
google                     # Bare word → contains "google"
```

---

## Keyboard Controls

[](https://github.com/matthart1983/netwatch#keyboard-controls)

|Key|Action|
|---|---|
|`1`–`8`|Switch tabs|
|`↑` `↓`|Navigate|
|`p`|Pause / resume|
|`r`|Force refresh|
|`R`|Arm / reset flight recorder|
|`F`|Freeze current incident window|
|`E`|Export incident bundle|
|`/`|Filter (Packets)|
|`c`|Start/stop capture (Packets)|
|`s`|Sort / stream view|
|`w`|Export to .pcap|
|`T`|Traceroute|
|`W`|Whois lookup|
|`t`|Cycle theme|
|`,`|Settings|
|`?`|Help|
|`q`|Quit|

**Full keybinding reference**

### Connections

[](https://github.com/matthart1983/netwatch#connections)

|Key|Action|
|---|---|
|`s`|Cycle sort column|
|`Enter`|Jump to Packets with connection filter|
|`T`|Traceroute to remote IP|
|`W`|Whois lookup|
|`e`|Export connections to JSON + CSV|
|`g`|Toggle GeoIP column|

### Packets

[](https://github.com/matthart1983/netwatch#packets)

|Key|Action|
|---|---|
|`c`|Start/stop capture|
|`R`|Arm / disarm flight recorder|
|`F`|Freeze incident window|
|`E`|Export incident bundle|
|`i`|Cycle capture interface|
|`b`|Set BPF capture filter|
|`/`|Display filter|
|`s`|Stream view|
|`w`|Export .pcap|
|`x`|Clear packets|
|`m`|Bookmark packet|
|`n`/`N`|Next/prev bookmark|
|`f`|Auto-follow|
|`W`|Whois lookup for selected packet IPs|

### Stream View

[](https://github.com/matthart1983/netwatch#stream-view)

|Key|Action|
|---|---|
|`→` `←`|Filter A→B / B→A|
|`a`|Both directions|
|`h`|Toggle hex/text|
|`Esc`|Close|

### Topology

[](https://github.com/matthart1983/netwatch#topology)

|Key|Action|
|---|---|
|`T`|Traceroute to selected host|
|`Enter`|Jump to Connections for host|
|`Esc`|Close traceroute overlay|

### Timeline

[](https://github.com/matthart1983/netwatch#timeline)

|Key|Action|
|---|---|
|`t`|Cycle time window (1m–1h)|
|`Enter`|Jump to Connections|

### Processes

[](https://github.com/matthart1983/netwatch#processes)

|Key|Action|
|---|---|
|`↑` `↓`|Navigate|
|`e`|Export connections to JSON + CSV|

### Settings

[](https://github.com/matthart1983/netwatch#settings)

|Key|Action|
|---|---|
|`↑` `↓`|Navigate settings|
|`Enter`|Edit selected setting|
|`←` `→`|Cycle theme|
|`S`|Save config|
|`Esc`|Close|

---

## Incident Bundle

[](https://github.com/matthart1983/netwatch#incident-bundle)

When the Flight Recorder is armed, NetWatch keeps a bounded rolling window of evidence. On freeze or export, it writes:

```
netwatch_incident_20260403_103501/
  summary.md
  manifest.json
  connections.json
  health.json
  bandwidth.json
  dns.json
  alerts.json
  packets.pcap   # present when packets were captured
```

This makes bug reports, incident reviews, and demos much easier: you keep the packet evidence and the operational context that explains it.

---

## Permissions

[](https://github.com/matthart1983/netwatch#permissions)

|Feature|`netwatch`|`sudo netwatch`|
|---|---|---|
|Interface stats & rates|✅|✅|
|Active connections|✅|✅|
|Network configuration|✅|✅|
|Health probes (ICMP)|❌|✅|
|Packet capture|❌|✅|

Degrades gracefully — features that need root show a clear message, never crash.

---

## Themes

[](https://github.com/matthart1983/netwatch#themes)

5 built-in themes with instant switching via `t`:

**Dark** (default) · **Light** · **Solarized** · **Dracula** · **Nord**

Theme changes apply immediately. Persist them from the Settings overlay with `S`.

---

## Configuration

[](https://github.com/matthart1983/netwatch#configuration)

NetWatch runs well with zero setup, but you can persist preferences for theme, default tab, refresh rate, capture interface, GeoIP database paths, packet-follow behavior, BPF filter, and alert thresholds.

netwatch --generate-config

That writes a starter config file to your platform config directory. You can also edit settings live in the app with `,` and save with `S`.

---

## How It Works

[](https://github.com/matthart1983/netwatch#how-it-works)

|Collector|Interval|macOS|Linux|
|---|---|---|---|
|Interface stats|1s|`netstat -ib`|`/sys/class/net/*/statistics`|
|Connections|2s|`lsof -i -n -P`|`/proc/net/tcp` + `/proc/*/fd`|
|Health probes|5s|`ping`|`ping`|
|Packets|Real-time|libpcap (BPF)|libpcap|
|GeoIP|On-demand|MaxMind .mmdb / ip-api.com|MaxMind .mmdb / ip-api.com|

```
Raw bytes → Ethernet → IPv4/IPv6/ARP → TCP/UDP/ICMP → DNS/TLS/HTTP/DHCP/NTP
                                             ↓
                               Stream tracking · Handshake timing
                               Expert info · Payload extraction
```

---

## Related

[](https://github.com/matthart1983/netwatch#related)

**[ESSH](https://github.com/matthart1983/essh)** — If you manage the hosts you monitor, ESSH is built for the same workflow. Same TUI aesthetic, pure-Rust SSH client with concurrent sessions, live remote host diagnostics (CPU, memory, disk, processes — no agent install), fleet management, file transfer, and port forwarding. Connects where NetWatch observes.

---

## Contributing

[](https://github.com/matthart1983/netwatch#contributing)

Contributions welcome! See [CONTRIBUTING.md](https://github.com/matthart1983/netwatch/blob/main/CONTRIBUTING.md) for coding conventions and [WIKI.md](https://github.com/matthart1983/netwatch/blob/main/WIKI.md) for a current architecture guide.

## License

[](https://github.com/matthart1983/netwatch#license)

MIT