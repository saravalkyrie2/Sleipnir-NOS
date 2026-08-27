## Officially Supported Devices

"Officially" meaning: someone I can physically reach the serial console.

| Device | Persona | Silicon | CPU / Arch | Status |
|---|---|---|---|---|
| Edgecore/Accton **AS7712-32X** (32×100G QSFP28) | SwitchOS | Broadcom BCM56960 "Tomahawk" | Intel Atom C2538 / x86-64-v2 | ✅ **Supported** — the bench reference; everything in this README happened on it |
| Edgecore **AS4610-54P** (48×1G PoE+, 4×SFP+) | SwitchOS | Broadcom Helix4 | ARM Cortex-A9 / armhf | 🗺️ Planned — brings the armhf port, u-boot ONIE, and a PoE subsystem SONiC never gave it |
| Generic **x86-64** router (x86-64-v2+) | RouteROS | Your NIC's finest | x86-64 | ✅ Supported — kernel dataplane; VPP-class forwarding on the roadmap |
| VMware ESXi | RouteROS | Not everyones favorite... | x86-64 | ✅ Supported — kernel dataplane; VPP-class forwarding on the roadmap |
| **saivs** virtual switch (VM) | SwitchOS (dev) | Simulated | any | 🧪 Development only — where code goes before it's allowed near the Tomahawk |

Hardware note: build flags respect the humblest CPU in the fleet (x86-64-v2) — a lesson
learned when an over-optimized OpenSSL took `sshd` down on the exact box you need `sshd` to
reach. The Atom remembers. So do we.
