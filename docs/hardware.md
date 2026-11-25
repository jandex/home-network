# Hardware

Prices are Australian street prices checked in November 2026. They move. The point of the table is role and regret, not a quote.

## Core network

| Device | Role | Why this, not the cheaper lookalike | Approx. AUD |
|--------|------|--------------------------------------|-------------|
| NBN NTD | ISP handoff | Already in the premises. | — |
| TP-Link Omada ER605 v2 | Gateway. Firewall, inter-VLAN routing, DHCP, WireGuard | About $90. Does the job the Archer cannot. Firmware 2.3.2 or later, so WireGuard is present. ER7206 is unnecessary on a 500 Mbps plan. | ~$90 |
| TP-Link Omada ES205GP | 5-port switch, 4× PoE+, 65 W budget, 802.1Q VLANs | Tags VLANs and powers one AP. SG2008P has a fuller Omada feature set and more ports; it is not required for three VLANs and one PoE AP. The switch is not the firewall. | ~$80–$110 |
| TP-Link Omada EAP650-Desktop | Hallway Wi-Fi 6 AP, PoE, 4× GbE | Desktop body, so no wall plate and no drilling. Downlink ports are the whole reason it is here: upstairs PC on one ETH, TV on another. | ~$160–$330 |
| TP-Link Archer AXE5400 | Upstairs access point only | Already owned. Fed from the same VLAN 10 cable as the PC. It is not the gateway. | Already spent, ~$230–$300 |
| Line-interactive UPS, ~900VA | Ride through a short outage | One circuit in a rental. Feeds the NBN box, ER605, ES205GP, and Mini PC only. | ~$150–$250 |

PoE check: the EAP650-Desktop draws about 17 W with PoE-out off. The ES205GP has a 65 W budget. One desktop AP leaves a wide margin. Leave PoE-out off. Do not hang a second PoE camera off this switch without redoing the sum.

Cabinet load on the UPS is roughly 70–80 W, on the order of half an hour to an hour. Home Assistant NUT shuts the Mini PC down with battery still left. The TV, heat pump, and washer stay off the UPS.

## Already owned, and what went wrong

The Archer AXE5400 was bought first, as the main router, on a 500 Mbps plan.

It is a good Wi-Fi 6E radio. It is a poor policy router. It has a guest network. It does not have 802.1Q VLANs plus a firewall between them. That was the requirement (IoT and the TV must not reach the automation host). Guest Wi-Fi is not that requirement.

The fix is not a second router behind it. That is double NAT: Home Assistant discovery breaks, port forwards get confusing, and you still do not have real VLANs.

The fix is:

1. Put the ER605 on the NBN NTD.
2. Factory-reset the Archer and set it to access-point mode.
3. Feed it from the upstairs end of the EAP650 VLAN 10 cable, next to the PC, so a second long run from the cabinet is never pulled.

That cable is access VLAN 10. The Archer can extend the home SSID only. It cannot put a projector on VLAN 30. The projector joins the EAP650 media SSID, or it sits on VLAN 10 and is treated as trusted.

Money outcome:

| Item | AUD | Verdict |
|------|-----|---------|
| Archer, spent before the design was finished | ~$230–$300 | Sunk. Still useful as an AP, so it is not binned. |
| ER605, required anyway once VLANs were the goal | ~$90 | This is the extra outlay caused by buying the router first. |
| A plain upstairs AP, if the Archer had never been bought | ~$100–$160 | What the upstairs radio "should" have cost. |

The Archer was the expensive way to buy an access point. The $90 gateway was always going to be required for this design. Buying the Wi-Fi box before writing the VLAN rules is the mistake, not the brand.

## Automation host

| Device | Role | Notes |
|--------|------|-------|
| Mini PC (N150-class, 16 GB RAM, 500 GB–1 TB SSD) | Home Assistant OS, AdGuard, Omada controller | Wired to the ES205GP on VLAN 10, address 10.10.10.10. No monitor required after install. |
| Home Assistant Connect ZBT-2 | Zigbee (and Thread) coordinator | USB to the Mini PC. Radio coverage is from wherever that USB port is. It does not use the EAP650. |
| Raspberry Pi 4/5 + USB mic + small speaker | Voice satellite | Wyoming protocol back to Home Assistant. VLAN 10. Not the brain. |

A monitor, keyboard, and mouse are only needed to install Home Assistant OS. After that, the browser on the upstairs PC is the console. The upstairs PC is 10.10.10.20, cabled from an EAP650 access port on VLAN 10, not from the switch.

## IoT and media (not all bought yet)

| Device | Network | Why it is not on VLAN 10 |
|--------|---------|--------------------------|
| Reolink solar wireless camera | VLAN 20 Wi-Fi | No cable path through the sealed glass door. No drilling into brick. Alerts go through Home Assistant, then to the phone. |
| Roomba 675 | VLAN 20 Wi-Fi | Cloud schedule must keep working while you are away. It does not get a path to the PC. |
| Door, leak, and light sensors | Zigbee | Local. No IP address at all. |
| TCL heat pump | Zigbee IR blaster | The unit has no useful local API. IR is the control path. |
| TCL washing machine | Vendor cloud, if anything | Most of these do not expose a local API. Treat "automation" as best-effort. |
| Smart TV | VLAN 30, wired from an EAP650 ETH port | TVs are high-risk LAN clients. Phones on VLAN 10 still cast, via mDNS only. |
| Console, projector | VLAN 30 | If they are wireless, they use the media SSID on the EAP650, not the upstairs Archer. |

## Cabinet and cable

| Item | Choice |
|------|--------|
| Cabinet | Freestanding, wood, side-table size. Not a wall-mount rack. The rental cannot take screws, and a rack in the hallway narrows it. |
| What goes in it | NBN NTD (if it can be moved or the cabinet sits beside it), ER605, ES205GP, Mini PC, one power board, the UPS. |
| Cable | Cat6 solid for the upstairs run. Cat6 stranded patch leads at the ends. Cat7 and Cat8 buy nothing on a 1 Gbps switch. |
| Power | One quality board in the cabinet, plugged into the UPS. A second board only if the upstairs PC and Archer need their own outlet. |
| Cable management | Low-tack raceway or existing skirting gaps. Avoid permanent adhesive on paint. |

## Buy order, if spending slowly

1. ER605 — without it, every later VLAN is theatre.
2. ES205GP + Cat6 — power and tags for the AP, and the long upstairs lead.
3. EAP650-Desktop — hallway Wi-Fi and the two downlink ports (PC and TV).
4. UPS — NBN box, gateway, switch, Mini PC.
5. Reuse the Archer upstairs, in AP mode, on the PC's VLAN 10 cable. Do not buy another router.
6. Mini PC + ZBT-2 — once the VLANs pass the ping tests in [architecture.md](architecture.md).
7. Voice satellite, then cloud voice keys. Voice is the last layer, not the first invoice.
