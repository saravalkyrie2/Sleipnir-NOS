# Sleipnir NOS

*A network operating system, written in C#, on Arch Linux. Yes, really. It's going fine, thanks for asking.* 

This is a staging repo for the Nordlance Sleipnir NOS - Moonshot project (Circa 2023+)

Sleipnir is a from-scratch NOS with one shared core and two personalities:

- **Sleipnir-RouteROS** — x86 routers, kernel dataplane (VPP-class dataplane on the roadmap).
- **Sleipnir-SwitchOS** — Real ASIC switches, driven **SAI-direct** (`libsaibcm`, no SONiC software stack in sight). The SONiC replacement, for those who like to have a life without endless phone calls at 3AM.

Named for Odin's eight-legged horse, because a switch with 32 ports of 100G deserves a mount with
more legs than average, and because every infrastructure project is required to have a
Norse or Germanic name.

## Why would you do this?

The VITM fleet ran VyOS-on-Arch (a steady diet of Debian-assumption bugs) and SONiC (a "do everything
everyone at a hyperscaler needs, eventually") distribution whose quirks made most of the newcomers squirm. 

Sleipnir keeps the *good* ideas —
config-DB architecture, commit/rollback model, orchestrating mature daemons
instead of rewriting BGP — and puts them on one deliberately small, deliberately owned core:

- **Config model**: candidate → validate → commit → rollback, with CONFIG/APPL/STATE separation
  (SQLite, not Redis — this is an appliance, not a cluster).
  
- **Control plane**: a single .NET process. gRPC API, TACACS+→local auth (C# Native from scratch), interactive CLI with
  the `?`/Tab manners you expect from something that calls itself a NOS.
  
- **Dataplane seam**: the same binary drives `iproute2` (router) or Broadcom(For now) SAI via hand-built
  P/Invoke bindings (switch). The bindings are *measured, never guessed* — ask us about the
  48-vs-72-byte struct incident sometime. Bring snacks, and maybe caffine.

## Does it actually work?

Unf***ing believably yes! - (When Sarah says "moonshot" she means no one should or would be delusional enough to even consider it...) Ben@OV1 North

As of Aug 2026 -
On the bench right now, an Accton AS7712-32X (Tomahawk, 32×100G) cold-boots unattended into a
serving control plane, restores its committed config, names its ports the same way the SONiC
fleet does (`Ethernet0…124`, lane-keyed from the platform port map), and brings up real 100G/40G
links — config → commit → ASIC → linkscan → carrier → kernel netdev, every layer observable
from `show interfaces`, `ip a`, the journal, and an audited in-CLI Broadcom diag shell
(`bcmshell`) for when you need to speak to the silicon directly.

Highlights of the current feature set:

| Thing | State |
|---|---|
| Commit/rollback config pipeline (interfaces, VLANs, speeds) | ✅ shipped |
| SAI switch bring-up on real Tomahawk hardware | ✅ shipped |
| KNET hostif netdevs with true carrier state | ✅ shipped |
| Lane-based port naming (SONiC-convention, breakout-ready) | ✅ shipped |
| `show capabilities`, `bcmshell`, startup reconciliation | ✅ shipped |
| CoPP, native LLDP/DCBX, STP, BGP-via-FRR, fan/SMART monitoring | 🔨 the roadmap is honest about being a roadmap |

The port-bring-up lessons learned the hard way (Broadcom speed changes bounce ports; vendor
diag names renumber themselves; the i2c bus must be unclaimed at ASIC init or nothing works)
are encoded in code comments at the exact lines where they'll ambush the next person.

- Trust But Verify, LLM or Human far too many people told me that the EdgeCore tweaks were not the reason SAI wouldn't start..... (Guess what manufacturers do thing for a genuine reason sometimes...)

## Architecture in one breath

Arch Linux base image (UEFI ISO or ONIE installer) → Kernel Patches and Tweaks → systemd runs one control-plane service →
EF Core/SQLite config stores → commit pipeline fans out to per-subtree appliers → appliers
program the kernel or the ASIC → state reads back for `show`. Mature protocol daemons (FRR,
chrony, etc…) get orchestrated at arm's length — rendered config, atomic reload, state
read-back — rather than reimplemented. Protocol engines we *do* own (LLDP/DCBX, STP) live in
`Sleipnir.Core.*` libraries, because the upstream options either don't exist, apply their
results somewhere the ASIC can't see, or tell you in their README not to use them in production.

Speaking of "or tell you in their README not to use them in production."

DO NOT USE This in production - yet. 

## Standing on shoulders

FRR, the OCP SAI project, sonic-buildimage's platform data, OpenHardwareMonitor (MPL-2.0) for
hardware-monitoring wisdom, and the Broadcom SDK — a 530 MB shared object that, at the moment
of truth, syncs port speed to the kernel by shelling out to `ethtool`. We aspire to that level
of pragmatism.

## Status

Bench-alpha, developed against real hardware, by a infrastructure engineer who doesn't understand the question "Why do this?" or "Brick walls are there for a reason"  with a very large serial
console habit. Not accepting production deployments; enthusiastically accepting the lessons of
everyone who tried this before me.

TLDR:

I got fed up with running Debian/Ubuntu from the dark ages or with 1000s of lines of Python, without coherent documentation designed for operators, not just contributors/developers that didn't even have vlan add range.. 

So i started building one.
