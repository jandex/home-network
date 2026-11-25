# VLANs and firewall

VLANs without a firewall are coloured cables. They do not stop a TV, a camera, or a vacuum from scanning the automation host.

## Where each job is done

| Job | Box | Not this box |
|-----|-----|----------------|
| Tag frames with VLAN 10 / 20 / 30 | ES205GP, and the EAP650 SSIDs and ETH ports | Archer AXE5400 |
| Route between VLANs | ER605 | The switch. It is Layer 2. |
| Permit or deny those routes | ER605 firewall | The switch. ES205GP has no inter-VLAN policy. |
| DHCP and DNS addresses | ER605 hands them out | Clients must not keep the ISP resolver |
| Ad blocking | AdGuard on the Mini PC, 10.10.10.10 | The gateway, except as the fallback resolver |
| Wi-Fi encryption | EAP650, and the Archer once it is an AP | The gateway |
| Remote access | WireGuard on the ER605, 10.10.40.0/24 | Port 8123 on the public IP |

The Archer stays in the design as a radio on VLAN 10. It is not in the policy path.

## VLAN map

| VLAN | Name | Gateway | DHCP | Members |
|------|------|---------|------|---------|
| 10 | Trusted | 10.10.10.1/24 | ER605 | Mini PC .10, PC .20, EAP650 .2, switch .3, phones, Archer management, voice satellite |
| 20 | IoT | 10.10.20.1/24 | ER605 | IoT SSID on the EAP650. No wired IoT in this house. |
| 30 | Media | 10.10.30.1/24 | ER605 | TV on an EAP650 access port. Media SSID for a console or projector. |
| — | WireGuard | 10.10.40.0/24 | ER605 | Phone and laptop away from home. Not a VLAN on the switch. |

Do not use 192.168.0.0/24 or 192.168.1.0/24. Those collide with café Wi-Fi when the tunnel is up. Do not use the same /24 on two VLANs.

IPv6 stays off until the same three zones exist in v6. A v4-only ruleset is a bypass.

## Ports on the EAP650

| Port | Mode | Device |
|------|------|--------|
| Uplink | Trunk, VLANs 10, 20, 30. PoE-in. | ES205GP |
| ETH | Access VLAN 10, untagged | Long cable to the upstairs PC. Archer AP may share this VLAN. |
| ETH | Access VLAN 30, untagged | Smart TV |
| Last ETH | Unused | PoE-out stays off |

The Mini PC and the voice satellite are access VLAN 10 on the ES205GP, not on the AP.

## Rules

Applied on the ER605. First match wins. Omada ACLs are direction-specific: a rule one way does not imply the reply. Replace the names with the ER605 interface names after the VLANs exist.

```text
1  ALLOW   established / related
2  ALLOW   VLAN10 → VLAN20          source: Mini PC
3  ALLOW   VLAN10 → VLAN30          source: Mini PC, and phones that cast
4  ALLOW   VLAN20 → 10.10.10.10     UDP/TCP 53 only
5  ALLOW   VLAN30 → 10.10.10.10     UDP/TCP 53 only
6  ALLOW   VLAN20 → WAN             UDP 123, TCP 443
7  ALLOW   VLAN30 → WAN             UDP 123, TCP 80, TCP 443
8  ALLOW   VLAN10 → WAN
9  ALLOW   VLAN10 ↔ VLAN30          UDP 5353 mDNS only
10 DENY    VLAN20 → VLAN10
11 DENY    VLAN30 → VLAN10
12 DENY    VLAN20 → VLAN30
13 DENY    VLAN30 → VLAN20
14 DENY    WAN → LAN                except WireGuard UDP
15 DENY    any → any
```

Rule 15 matters. A gateway that allows LAN to LAN by default will undo rules 10 to 13.

DNS primary for every VLAN is 10.10.10.10 (AdGuard). Secondary is 9.9.9.9, so the house still resolves if the Mini PC is off. Filtering is off during that fallback. VLAN 10 gets the privacy lists. VLAN 20 and VLAN 30 get malware-only lists, with iRobot and the streaming names left alone.

What this means in practice:

| From | To Mini PC | To the internet | To the TV |
|------|------------|-----------------|-----------|
| Mini PC | — | Yes | Yes |
| Phone on VLAN 10 | Yes, same VLAN | Yes | Yes, for casting |
| IoT camera / Roomba | DNS only, port 53 | Yes, 443 and NTP | No |
| Smart TV | DNS only, plus mDNS | Yes | — |

Home Assistant reaches a camera because the Mini PC starts the session. The camera cannot turn around and open a session to the Mini PC.

If the TV is discovered and the cast dies on tap, the TV is opening a session back toward the phone. Add a return allow from VLAN 30 to VLAN 10 for the cast ports only. Do not open VLAN 30 to VLAN 10 in general.

## WireGuard

| Setting | Value |
|---------|--------|
| Listen | One UDP port on the WAN. Nothing else is forwarded. |
| Tunnel | 10.10.40.0/24 |
| Phone AllowedIPs | 10.10.10.0/24, 10.10.20.0/24, 10.10.30.0/24, 10.10.40.0/24 |
| VLAN 20 and VLAN 30 default route | Straight out the NBN. Never into the tunnel. |

Alerts arrive without the tunnel, because the Mini PC pushes them out. Opening Home Assistant or the camera needs the tunnel.

## What not to do

- Do not put a second NAT router on VLAN 20 "for safety". The ER605 already isolates that VLAN. A second router adds double NAT and hides the devices from Home Assistant.
- Do not port-forward Home Assistant to the static public IP. Opting out of CGNAT makes inbound scans succeed if a port is open.
- Do not give the IoT SSID the same password as the trusted SSID.
- Do not put the switch or AP admin page on VLAN 20 or VLAN 30.
- Do not point the PC's long cable at a trunk port. Untagged VLAN 10, or the PC lands on the native VLAN by accident.

## SSID plan on the EAP650

Three is enough.

| SSID | VLAN | Band | Used by |
|------|------|------|---------|
| `home` | 10 | 2.4 + 5 | Phones, laptops. Not IoT. |
| `iot` | 20 | 2.4 is enough | Roomba, camera, washer, plugs. Client isolation on. |
| `media` | 30 | 5 GHz | Console, projector, a streamer that is not the wired TV. Isolation off. |

The upstairs Archer, in AP mode, broadcasts `home` only. It is on an access VLAN 10 cable. Do not make it broadcast `iot` or `media`. Two radios on `home` plus VLAN 10 is normal. Two radios on the same SSID and different VLANs is a bug.

## QoS

On the ER605, set bandwidth control to the real plan (500 Mbps down, and whatever the upload actually is). Priority, if the firmware offers it:

| Class | Traffic |
|-------|---------|
| Highest | Voice-satellite audio |
| Normal | Mini PC, browsing, the upstairs PC |
| Lower | VLAN 30 streaming, camera cloud upload |

## Test sheet

Run from a device joined to each SSID, and from the wired PC. Record the date. This page is not done until the fails really fail.

| Test | VLAN 10 | VLAN 20 | VLAN 30 |
|------|---------|---------|---------|
| Ping 1.1.1.1 | Pass | Pass | Pass |
| Ping Mini PC | Pass | **Fail** | **Fail** |
| Browse `http://10.10.10.10:8123` | Pass | **Fail** | **Fail** |
| DNS lookup via 10.10.10.10 | Pass | Pass | Pass |
| Ping the TV | Pass for a casting phone | **Fail** | Pass |
| Cast phone to TV | Pass | **Fail** | — |
| Home Assistant can toggle a test IoT entity | Pass from the Mini PC | — | — |
| Roomba still reaches iRobot | — | Pass | — |

If a "Fail" passes, stop and fix the ER605 rule before adding more devices.
