# FRRouting (FRR)

The minimum FRRouting version to support `extended-nexthop` (v4 with v6 nexthops) is 7.0.0.

The [documentation](https://docs.frrouting.org/en/latest/bgp.html#clicmd-neighbor-PEER-capability-extended-nexthop) talks about `extended-nexthop` to allow bgp to negotiate the extended-nexthop capability with it’s peer and allow BGP to install v4 routes with v6 nexthops.

## Minimum Software Version

The minimum FRRouting version to support RFC5549 is 7.0.0. This version introduced the `extended-nexthop` term.
We are running the test with FRRouting 8.2.0.

## Configuration

Configuration for route-server:

```
router bgp 133339
 no bgp default ipv4-unicast
 neighbor 2400:9c80:0:171::2 remote-as 100
 neighbor 2400:9c80:0:171::2 description Example-AS100
 neighbor 2400:9c80:0:171::2 capability extended-nexthop
 !
 address-family ipv4 unicast
  neighbor 2400:9c80:0:171::2 activate
  neighbor 2400:9c80:0:171::2 next-hop-self
  neighbor 2400:9c80:0:171::2 soft-reconfiguration inbound
  neighbor 2400:9c80:0:171::2 route-server-client
  neighbor 2400:9c80:0:171::2 route-map in-as100 in
  neighbor 2400:9c80:0:171::2 route-map out-as100 out
  neighbor 2400:9c80:0:171::2 attribute-unchanged next-hop
 exit-address-family
 !
 address-family ipv6 unicast
  neighbor 2400:9c80:0:171::2 activate
  neighbor 2400:9c80:0:171::2 next-hop-self
  neighbor 2400:9c80:0:171::2 soft-reconfiguration inbound
  neighbor 2400:9c80:0:171::2 route-server-client
  neighbor 2400:9c80:0:171::2 route-map in-as100 in
  neighbor 2400:9c80:0:171::2 route-map out-as100 out
  neighbor 2400:9c80:0:171::2 attribute-unchanged next-hop
 exit-address-family
exit
```

Important lines are:

1. `capability extended-nexthop` - this is not default in this case!

## Testing

### Are the routes getting accepted?

Yes:

```
# show ip bgp neighbors 2400:9c80:0:171::2 received-routes 
BGP table version is 2, local router ID is 1.1.1.1, vrf id 0
Default local pref 100, local AS 133339
Status codes:  s suppressed, d damped, h history, * valid, > best, = multipath,
               i internal, r RIB-failure, S Stale, R Removed
Nexthop codes: @NNN nexthop's vrf id, < announce-nh-self
Origin codes:  i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

   Network          Next Hop            Metric LocPrf Weight Path
*> 10.100.100.0/24  2400:9c80:0:171::2
                                             0             0 100 i

Total number of prefixes 1
```

### Is the next-hop address resolution working?

Yes:

```
# show ip route 10.100.100.0/24
Routing entry for 10.100.100.0/24
  Known via "bgp", distance 20, metric 0, best
  Last update 00:10:22 ago
  * fe80::e8d:e8ff:fe9e:0, via eth0, weight 1
```

### What does traceroute look like?

The route-server will not be visible in the traceroute output from peers, because it is not in the forwarding path.

### What happens if peer is not configured to use RFC8950 next hops?

If extended next-hop cannot be negotiated, neither IPv4 routes can be received not advertised.