# Home network

Locked design. Three VLANs, PC cabled from the EAP650, Roomba cloud schedule, camera alerts through Home Assistant, DNS fallback, one UPS.

The Archer AXE5400 is replaced by an Omada ER605 v2 and removed from the path. It stays in the house as an upstairs access point on the same VLAN 10 cable as the PC. Leaving it in front of the ER605 creates a second NAT and wastes the static public IPv4.

## Documents

| Doc | What it is |
|-----|------------|
| [docs/architecture.md](docs/architecture.md) | Why each box is where it is |
| [docs/hardware.md](docs/hardware.md) | Parts, prices, buy order, cabinet and cable |
| [docs/vlans-and-firewall.md](docs/vlans-and-firewall.md) | Subnets, rules, SSIDs, test sheet |
| [docs/voice-assistant.md](docs/voice-assistant.md) | Assist pipeline, what Claude is not allowed to do |
| [docs/lessons-learned.md](docs/lessons-learned.md) | What the first router got wrong |

The rest of this page is the locked design. The docs above are the longer write-up: why the Archer was the wrong first buy, the buy order, the firewall test sheet, and the voice pipeline.
