# Huawei VRP

The Huawei NetEngine 8000 series partially supports IPv6 next-hops for IPv4 routes.

As of this writing Huawei VRP operating system does support receiving IPv4 routes with IPv6 next-hop. The routes may be accepted into the Loc-RIB, but **not** into the RIB or FIB.

Sending IPv4 routes with IPv6 next-hop works fine.

Static routes with IPv6 next-hops are working fine though.

## Minimum Software Version

RFC8950 support was added in VRP version V800R024. This version does **not** support BGP routes going into the RIB/FIB. It may be used as route reflector though.

Huawei R&D is working on full support and plans to release this 2nd half of 2027. You should get in contact with your sales representative. You mask ask for "RFC8950 support for inter-connection / SAFI=1".

## Configuration

```
interface 25GE0/5/22
 undo shutdown
 ipv6 enable
 ipv6 address 2A01:6421::1/64
 undo dcn

bgp 65501       
 peer 2A01:6421::2 as-number 65502
 #
 ipv4-family unicast
  network 10.45.1.0 255.255.255.0
  valid-route ipv6-nexthop enable
  peer 2A01:6421::2 enable
  peer 2A01:6421::2 next-hop-local
 #
 ipv6-family unicast
  peer 2A01:6421::2 enable
  peer 2A01:6421::2 next-hop-local
```

The most important line is: `valid-route ipv6-nexthop enable` -- this enables accepting IPv4 routes with IPv6 next-hop into the Loc-RIB.

The Huawei BGP implementation enables Extended Next-hop capability automatically -- no additional setting necessary.

If `next-hop-local` is enabled for the peer's IPv4 address-family, IPv4 are being advertised using the devices IPv6 address as next-hop.

Without configuring an IPv4 address on the interface the interface will be reported "Protocol down", but will forward IPv4 packets. It is possible to use e.g. `ip address unnumbered interface LoopBack0` though.

## Testing

### Are the routes getting accepted?

Example:

```
<HUAWEI>display bgp routing-table

 Total Number of Routes: 2
        Network            NextHop                       MED        LocPrf    PrefVal Path/Ogn

 *>     10.45.1.0/24       0.0.0.0                        0                     0       i
 *>     10.77.4.0/24       2A01:6421::2                                         0      65502?

<HUAWEI>display bgp routing-table 10.77.4.0 24
 
 BGP local router ID : 4.1.2.4
 Local AS number : 65501
 Paths:   1 available, 1 best, 1 select, 0 best-external, 0 add-path
 BGP routing table entry information of 10.77.4.0/24:
 From: 2A01:6421::2 (6.7.8.9)
 Route Duration: 48d00h13m42s
 Direct Out-interface: 25GE0/5/22
 Original nexthop: 2A01:6421::2
 Qos information : 0x0
 AS-path 65502, origin incomplete, pref-val 0, valid, external, best, select, pre 255
 Not advertised to any peer yet

```

### Is next-hop address resolution working?

No (not yet):

```
<HUAWEI>display ip routing-table 10.77.4.0 24
```

### What does traceroute look like?

yet unknown

### What happens if peer is not configured to use RFC8950 next hops?

If extended next-hop cannot be negotiated, neither IPv4 routes can be received nor advertised. Status can be checked:

```
<HUAWEI>display bgp peer 2A01:6421::2 verbose | incl Extended
 Extended Next Hop Encoding capability has been enabled
```

### What happens if peer sets an IPv4 address on the peering interface?

yet unknown
