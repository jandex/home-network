# Lessons learned

Written so the next design starts from the constraint, not from the router aisle.

## The Archer AXE5400 was the wrong first purchase

It was bought as the main router for a 500 Mbps NBN plan, before the isolation requirement was written down.

What it is good at:

- Wi-Fi 6E, including upstairs coverage for a far bedroom
- Access-point mode, so it can be reused instead of thrown out

What it cannot do:

- 802.1Q VLANs with a real firewall between them
- Be both the policy router and "good enough" once IoT and a Smart TV are on the LAN

A guest SSID was the workaround on the table. It is not the same control. Guest clients still sit on a feature the vendor can change, and there is no clean way to let only the Home Assistant host talk to them while everyone else is denied.

### What it cost

Street prices in Australia, November 2026, rounded.

| Purchase | Approx. AUD | Should it have been first? |
|----------|-------------|----------------------------|
| Archer AXE5400 | $230–$300 (seen at $227) | No |
| Omada ER605, bought afterwards | ~$90 | Yes. This is the gateway. |
| Omada ES205GP | ~$80–$110 | Yes, once an AP needed PoE and VLAN tags |
| EAP650-Desktop | ~$160–$330 | Yes, for the hallway and the short downlinks |

The ER605 is not a punishment. It was always required. The waste is paying Wi-Fi-6E-router money for a box that ended up as an access point. A plain upstairs AP would have been closer to $100–$160. The Archer still earns its keep upstairs, on the same VLAN 10 cable as the PC, because a second long pull from the cabinet was rejected. That reuse is why the box stays in the diagram. It is not why it was a good first buy.

Rule for the next build: write the firewall table, then buy the gateway that can implement it, then buy radios.

## Other calls that held up

| Call | Why it stays |
|------|----------------|
| No second "IoT router" | Double NAT. Home Assistant stops seeing devices. The ER605 already has the deny rules. |
| Mini PC on VLAN 10, not on VLAN 20 | The controller has to be the one allowed *in*. If it lives on the untrusted VLAN, a bug there is a bug next to the keys. |
| PC cabled from the EAP650, not the switch | One long run, from the hallway AP. The cabinet does not grow a second home-run upstairs. |
| ES205GP instead of SG2008P | Enough PoE and enough VLANs. The missing features were switch features, not firewall features. Buying the bigger switch would not have fixed the Archer. |
| Camera stays wireless and solar | Sealed glass door, no brick drilling, rental. A PoE camera with nowhere to break out is not a better camera. Alerts come from Home Assistant, not from a port forward. |
| Zigbee for sensors, Wi-Fi VLAN for cloud junk | A door sensor does not need an IP address. A Roomba does, and it does not get VLAN 10. Its cloud schedule has to keep working. |
| ZBT-2 next to the Mini PC | USB. The hallway is a better radio spot only if the host is there. A long USB extension is how you get a flaky coordinator. |
| Static IP, CGNAT off | The WireGuard endpoint. Dangerous if port 8123 is forwarded. The opt-out is not a security feature by itself. |
| Subnets in 10.10.0.0/16 | 192.168.0.0/24 and 192.168.1.0/24 collide with other people's routers when the laptop is away. |
| AdGuard on the Mini PC, Quad9 as fallback | Privacy lists on VLAN 10 only. If the Mini PC is dark, the house still resolves. |
| mDNS only between VLAN 10 and VLAN 30 | Casting was a requirement. A reflector onto VLAN 20 was not. |
| Claude as conversation agent only | Speech in, intent, speech out. MCP was the version where the model becomes an administrator. That failed the "would I do this in a station equipment room" test. |
| Cat6, not Cat7 or Cat8 | The switch is gigabit. The cable grade was not the bottleneck. |
| One UPS, not two feeds | A rental has one circuit. The UPS covers NBN, gateway, switch, and Mini PC. |

## What is still open

These are not done until they are tested. The docs should not pretend otherwise.

| Item | State |
|------|--------|
| ER605 deny rules, proven with pings | Not live until the test sheet in [vlans-and-firewall.md](vlans-and-firewall.md) is filled in |
| EAP650 port VLANs (PC on VLAN 10, TV on VLAN 30) | Design only |
| Archer reset into AP mode, on that VLAN 10 cable, and checked for a second DHCP server | Easy to get wrong. If the Archer still hands out addresses, the network will be weird in a way that looks like "Wi-Fi is flaky". |
| Filipino STT on the actual pipeline | Needs a recorded test, not a datasheet |
| ElevenLabs latency on the Pi speaker | Needs a stopwatch. If it is over a couple of seconds, Azure or a shorter reply prompt is the lever. |
| UPS on NBN box, gateway, switch, and Mini PC | Specified, not bought. |
| Home Assistant backups off the Mini PC | Not done. A dead SSD ends the project. |
| TCL washer | Probably not locally automatable. Do not buy hardware for it until a local API is confirmed. |

## How this reads on a CV

The story is not "I own an Archer". The story is:

- Requirement: untrusted devices must not reach the automation host, and the TV must still accept a cast.
- First product failed the isolation requirement.
- Replaced the policy layer, kept the radio, cabled the PC from the hallway AP instead of from the cabinet, and listed the tests that prove the deny rules.

That is the paragraph worth putting above the repo link.
