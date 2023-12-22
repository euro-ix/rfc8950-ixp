# Config Examples

Put config examples for different platforms and routing daemons in this directory.

## Findings So Far

| Vendor | Platform | Software Version | Notes |
| ------ | -------- | ---------------- | ----- |
| Arista | EOS | 4.22.1F | Some support already in 4.17 |
| Cisco | IOS XR | [7.3.3](https://www.ausnog.net/sites/default/files/ausnog-2022/presentations/ausnog_2022-day2-07-cooper_lees-who_needs_arp_v4_via_v6.pdf) | |
| CZNIC  | Bird     | 2.0.8  | RIB-only since 2.0.0; Linux kernel version 5.2 required |
| Exa | ExaBGP | 4.1.0 | Cannot program Linux netlinks for RFC5549 |
| Juniper | Junos | 17.3R1 | |
| Mikrotik | ROS | not supported | |
| NetDEF  | FRR      | 7.0.0 | Linux kernel version 5.2 required |
| RSSF | OpenBGPd | not supported | on the [roadmap](https://www.rssf.nl/roadmap) |

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
 * What happens if peer is not configured to use RFC8950 next hops?
   - Errors? Warnings? Alerts?
   - Blackholing?
   - Turn down session as a fail safe?
   - Other misconfiguration cases?
 * What happens if peer sets an IPv4 address on the peering interface?
   - Will the next-hops still be IPv6?
   - Will traceroutes now use this address instead of a loopback?
   - Could ISPs set up a /32 address on the interface to have a specific name show up in traceroutes?
