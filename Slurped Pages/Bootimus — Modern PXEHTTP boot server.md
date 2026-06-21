---
link: https://bootimus.com/
excerpt: Self-contained PXE and HTTP boot server. Single binary. Zero config. 50+ distros out of the box.
slurped: 2026-06-21T10:36
title: Bootimus — Modern PXE/HTTP boot server
tags:
  - linux
  - pxe
---

v1.x · apache 2.0 go · iPXE · sqlite/postgres

## PXE boot,  
without the pain.

Self-contained PXE and HTTP boot server. One binary. Zero config. Built-in proxyDHCP so you never touch your router. 50+ distros detected automatically.

pts/0

bootimus — quickstart

$ docker run -d --name bootimus \
    --cap-add NET_BIND_SERVICE \
    -p 67:67/udp -p 69:69/udp \
    -p 8080:8080/tcp -p 8081:8081/tcp \
    -v $(pwd)/data:/data \
    garybowers/bootimus:latest

$ docker logs bootimus | grep Password
admin: Password: 7f3a-plum-swift-echo

$ open http://localhost:8081

// features

## Everything a modern netboot setup should be.

Not a fork of 15-year-old Perl scripts. Not a wrapper around dnsmasq. A proper server, written in Go, with batteries included.

Go binary with embedded iPXE, web UI, SQLite, and all assets. No runtime deps. Scp it and run.

Answers PXE on UDP/67 without touching your existing DHCP. Zero router reconfig. Drop in on any LAN.

Automatic kernel/initrd extraction for Ubuntu, Debian, Arch, Fedora, NixOS, Alpine, FreeBSD, Windows (wimboot), and more.

Assign specific images per MAC. Auto-discover new clients on first PXE. Promote leases to static when ready.

GParted, Clonezilla, Memtest86+, SystemRescue, ShredOS, netboot.xyz. Enable from the UI, they show up in the menu.

Token auth with bcrypt. Optional LDAP/AD backend with group-based admin. Local accounts stay as fallback.

Everything the UI does is an API call. Script boot assignments, scans, WOL triggers. Live log stream over SSE.

Multi-arch Docker (amd64/arm64), static binary, or a 2GB Alpine-based appliance image you can flash to USB.

Drop autounattend.xml, kickstart, preseed, or cloud-init in. Attach to an image as the default, override per client. Bootimus stages it at boot — no clicks, no setup wizard.

// how it works

## The lifecycle of a PXE boot.

Client sends DHCPDISCOVER. Bootimus answers PXE details via proxyDHCP while your normal DHCP still hands out the IP. iPXE loads over TFTP, chains to HTTP, fetches the menu. User picks an image. Kernel and initrd stream from the server. Done.

pts/0

pxe boot trace — ubuntu-24.04

[dhcp]  → DHCPDISCOVER from b4:2e:99:01:5f:a3 (no PXE options from primary DHCP)
[proxy] ← DHCPOFFER-PXE: next-server=bootimus, filename=ipxe.efi
[tftp]  → RRQ ipxe.efi  (198 KiB, 14 ms)

[http]  → GET /menu.ipxe  200  2.1 KiB
[menu] ↳ 17 images · 3 groups · 6 tools
[menu] ↳ user selected: ubuntu-24.04-live-server

[http]  → GET /iso/ubuntu-24.04/vmlinuz  200  14 MiB · 612 MB/s
[http]  → GET /iso/ubuntu-24.04/initrd   200  76 MiB · 598 MB/s
[boot]  handoff ok · client booting

// transparency

## 100% open. Auditable end-to-end.

No proprietary blobs. No telemetry. No sneaky binary firmware vendored in. The whole stack is on GitHub under Apache 2.0 — clone it, audit it, fork it, fly your own.

- ✓
    
    **Single Go binary** · statically linked, ldd returns "not a dynamic executable". Reproducible builds from make release.
    
- ✓
    
    **No proprietary blobs** · embedded iPXE is upstream FOSS (GPL-2.0). No closed-source firmware shipped.
    
- ✓
    
    **No telemetry, ever** · zero call-home. Zero analytics. Zero "anonymous usage stats". Air-gapped LAN safe.
    
- ✓
    
    **Apache 2.0** · permissive licence. Use in commercial environments, ship internally, fork without strings.
    
- ✓
    
    **Vendored deps, all FOSS** · every transitive Go dependency is open source. go mod why any package.
    
- ✓
    
    **Bring your own bootloader** · don't trust the embedded iPXE? Drop your own signed binaries in. See below.
    

pts/0

bootimus version --verbose

$ bootimus version --verbose
bootimus      1.0.0
commit       8e87824 (clean)
go           1.23.4 linux/amd64
build        static · reproducible
licence      Apache-2.0

embedded
  ipxe        1.21.1+upstream  GPL-2.0
  proprietary 0 blobs
  telemetry   disabled (compile-time)

$ ldd ./bootimus
not a dynamic executable

$ sha256sum ./bootimus
7f3a9b0c…  bootimus

// bootloaders

## Swap iPXE for whatever you need.

Bootimus ships with embedded iPXE for every common arch. Need Microsoft-signed binaries for Secure Boot, a custom-themed iPXE, GRUB, syslinux, or your own internal-CA-signed loader? Drop a folder in data/bootloaders/, pick it from the UI, done. Missing files transparently fall back to the embedded set — never a broken boot.

### iPXE · UEFI x86_64

ipxe.efi · the default. Built from upstream master, embedded in the binary.

embedded · fallback

### iPXE · UEFI ARM64

ipxe-arm64.efi · for Raspberry Pi 4/5, Apple Silicon hosts, ARM servers.

embedded · fallback

### iPXE · Legacy BIOS

undionly.kpxe · for old kit that won't UEFI. Still relevant in 2026.

embedded · fallback

### Microsoft-signed shim

Drop a signed shimx64.efi + grubx64.efi in for Secure-Boot-enforced fleets. No firmware MOK enrolment needed.

custom · BYO

### Custom-themed iPXE

Compile your own iPXE with branding, custom menu colours, embedded scripts. Drop the .efi in.

custom · BYO

### GRUB / syslinux / pxelinux

Not iPXE? No problem. Anything that speaks TFTP and HTTP works. Bootimus just serves bytes.

custom · BYO

pts/0

bootloader sets — file fallthrough

$ tree /var/lib/bootimus/bootloaders
data/bootloaders
├── ipxe-builtin/          
│   ├── ipxe.efi
│   ├── ipxe-arm64.efi
│   └── undionly.kpxe
├── ipxe-secureboot/       
│   ├── shimx64.efi             (signed by Microsoft)
│   ├── grubx64.efi             (signed by Microsoft)
│   └── ipxe.efi                (signed by your CA)
└── ipxe-themed/
    └── ipxe.efi                (custom branding)

$ bootimus bootloaders use ipxe-secureboot
✓ active set: ipxe-secureboot
✓ falls back to ipxe-builtin for: ipxe-arm64.efi, undionly.kpxe

[tftp] RRQ shimx64.efi      → ipxe-secureboot/shimx64.efi
[tftp] RRQ ipxe-arm64.efi   → ipxe-builtin/ipxe-arm64.efi (fallback)

## Ready to stop babysitting tftpd?

Docker, bare metal, or flashable USB. Pick your poison.