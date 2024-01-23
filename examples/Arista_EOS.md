# Arista EOS

Arista EOS supports [IPv4 addressless forwarding](https://www.arista.com/en/support/toi/eos-4-17-0f/13784-ip-addressless-forwarding-changes-for-bgp-v6-nexthop-for-v4-routes) 
as well as [RFC 5549](https://www.arista.com/en/support/toi/eos-4-23-2f/14436-rfc-5549-ipv4-unicast-nlri-with-ipv6-next-hop-support) 
BGP signalling.

## Minimum Software Version

Arista EOS comes with two different BGP implementation: ribd and multi-agent. While ribd has support for RFC5549 since EOS 4.17.x,
multi-agent was enhanced to support it from EOS 4.22.1F onwards. Multi-agent model is default nowadays, so it recommended to use it.

## Configuration

```
service routing protocols model multi-agent

ip routing ipv6 interfaces

interface Ethernet1
   no switchport
   ipv6 address 2001:7f8:19:6::1:1/64

interface Loopback0
   ip address 10.0.0.1/32

router bgp 65001
   router-id 10.0.0.1
   no bgp default ipv4-unicast
   neighbor peer peer group
   neighbor peer remote-as 65002
   neighbor peer next-hop-self
   neighbor peer send-community standard large
   neighbor 2001:7f8:19:6::2:1 peer group peer
   !
   address-family ipv4
      neighbor peer activate
      neighbor peer next-hop address-family ipv6 originate
   !
   address-family ipv6
      neighbor peer activate
```

There are four important lines:

1. `ip routing ipv6 interfaces` to enable IPv4 forwarding on non-IPv4 interfaces.
2. At least one (loopback) interface with an IPv4 address within the VRF. This address will be used for answering ICMP unreachables or during traceroutes.
3. `neighbor peer next-hop-self` should be default for IXP connection, but it's even more important here.
4. `neighbor peer next-hop address-family ipv6 originate` to enable extended next-hop facility and to also send out IPv4 routes with IPv6 next-hop.

## Testing

### Are the routes getting accepted?

Example:

```
# sh ipv6 bgp peers 2001:7f8:19:6::2:1 received-routes
          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      10.1.2.0/24            2001:7f8:19:6::2:1    -       -          -       -       65002 ?
 * >      10.1.3.0/24            2001:7f8:19:6::2:1    -       -          -       -       65002 ?

# sh ip bgp 10.1.2.0/24 detail 
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65001
BGP routing table entry for 10.1.2.0/24
 Paths: 1 available
  65002
    2001:7f8:19:6::2:1 from 2001:7f8:19:6::2:1 (10.0.0.2)
      Origin INCOMPLETE, metric 0, localpref 100, IGP metric 0, weight 0, tag 0
      Received 02:50:52 ago, valid, external, best
      Rx SAFI: Unicast
 Advertised to 1 peer:
  peer-group ibgp:
    10.1.1.1            
```

### Is next-hop address resolution working?

Yes:

```
# sh ip route 10.1.2.0

 B E      10.1.2.0/24 [20/0]
           via 2001:7f8:19:6::2:1, Ethernet1

# sh ipv6 neighbors 
IPv6 Address                                  Age Hardware Addr    State Interface
2001:7f8:19:6::2:1                        0:00:21 0c00.31b1.b33b   REACH Et1
```

### What does traceroute look like?

The router is visible with its loopback interface IPv4 address.

### What happens if peer is not configured to use RFC8950 next hops?

If extended next-hop cannot be negotiated, neither IPv4 routes can be received not advertised. Status can be checked:

```
# sh ipv6 bgp peers 2001:7f8:19:6::2:1 | grep -A1 "Extended Next-Hop"
    Extended Next-Hop Capability:
      IPv4 Unicast: advertised and received and negotiated
```

In case "IPv4 addressless forwarding" is not enabled, the routes are being accepted and visible in the FIB, but not forwarded, i.e. blackholed.

```
# sh ip | grep "Interface"
IPv6 Interfaces Forwarding : None

# conf
# ip routing ipv6 interfaces

# sh ip | grep "Interface"
IPv6 Interfaces Forwarding : True
```

### What happens if peer sets an IPv4 address on the peering interface?

When an IPv4 address is configured on the IXP facing interface, this IP address is used in traceroutes.

Arista only supports /31 or larger IPv4 addresses to be assigned to ethernet interfaces. Alternatively IP unnumbered might be configured referring to an IP address on some other loopback interface. The next-hop remains IPv6 in the FIB.
