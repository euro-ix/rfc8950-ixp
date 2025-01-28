# Config Examples

Put config examples for different platforms and routing daemons in this directory.

## Findings So Far

Links in the **Platform** column go to config examples for that platform.
| Vendor | Platform | Software Version | Notes |
| ------ | -------- | ---------------- | ----- |
| Arista | [EOS](Arista_EOS.md) | 4.22.1F | Some support already in 4.17 |
| Cisco | IOS XE | not supported | IPv6 next hop support for VPN routes since 17.8.1 |
| Cisco | [IOS XR](Cisco_IOSXR.md) | [7.3.3](https://www.ausnog.net/sites/default/files/ausnog-2022/presentations/ausnog_2022-day2-07-cooper_lees-who_needs_arp_v4_via_v6.pdf) | |
| Cisco | NX-OS | ? | to be tested |
| CZNIC  | Bird1     | not supported | Bird1 does not and will not support RFC8950 |
| CZNIC  | [Bird2](Bird2.md)     | 2.0.8  | RIB-only since 2.0.0; Linux kernel version 5.2 required |
| CZNIC  | [Bird3](Bird2.md)     | 3.0.0  | Bird3 config wrt RFC8950 settings is identical to Bird2 |
| Exa | ExaBGP | 4.1.0 | Cannot program Linux netlinks for RFC5549 |
| Extreme Networks | IronWare, SLX-OS | not supported | verified with vendor |
| Juniper | [Junos](Juniper_JunOS.md) | 17.3R1 | |
| Mikrotik | ROS | [not supported](https://help.mikrotik.com/docs/display/ROS/Routing+Protocol+Overview) | (last checked: 7.18) |
| NetDEF  | [FRR](FRRouting.md)  | 7.0.0 | Linux kernel version 5.2 required |
| Nokia | [SR-OS](Nokia_SROS.md) | 20.2.R1 | |
| Nokia | [SR Linux](Nokia_SR-Linux.md) | 20.06 | tested on 22.6.4 |
| OSRG | [GoBGP](GoBGP.md) | supported for several years | no FIB integration tested |
| RSSF | OpenBGPd | not supported | on the [roadmap](https://www.rssf.nl/roadmap) |
| Edgecore | OCNOS | not supported in 1.3.8 | Awaiting further comment from OCNOS developers |
| Vyatta | VyOS | 1.2.2 | See FRR above ||

## Check List for Testing

### Configuration
 * Find out minimum software requirements
 * What's the configuration difference between IPv4 next-hops and RFC8950 next-hops over an IPv6 BGP session?
 * Configure EBGP peer that uses RFC8950 style next hops
 * Configure source address/interface for ICMP Unreachables, if possible
### Testing
 * Are the routes getting accepted?
 * Is next-hop address resolution working?
 * What does traceroute look like?
 * How do you add route filters for IPv4?
   - Is it possible to add a separate route filter for IPv4 and IPv6 routes?
 * What happens if peer is not configured to use RFC8950 next hops?
   - Errors? Warnings? Alerts?
   - Blackholing?
   - Turn down session as a fail safe?
   - Other misconfiguration cases?
 * What happens if peer sets an IPv4 address on the peering interface?
   - Will the next-hops still be IPv6?
   - Will traceroutes now use this address instead of a loopback?
   - Could ISPs set up a /32 address on the interface to have a specific name show up in traceroutes?
