# Cisco IOS XR

## Minimum Software Version

Presumably minimum version is IOS-XR 7.3.3 ([3rd-party source](https://www.ausnog.net/sites/default/files/ausnog-2022/presentations/ausnog_2022-day2-07-cooper_lees-who_needs_arp_v4_via_v6.pdf)).

## Configuration

```
interface GigabitEthernet 0/1/1
  ipv6 address 2001:db8:1::1/126

router bgp 64500
    neighbor 2001:db8:1::2
        remote-as 64496
        description Example
        enforce-first-as
        address-family ipv4 unicast
            maximum-prefix 5 95
            send-community-ebgp
            route-policy session-as64496-v4-in in
            route-policy session-as64496-v4-out out
            send-extended-community-ebgp
            next-hop-self
            remove-private-AS
            soft-reconfiguration inbound always
        !
        address-family ipv6 unicast
            maximum-prefix 5 95
            send-community-ebgp
            route-policy session-as41731-v6-in in
            route-policy session-as41731-v6-out out
            send-extended-community-ebgp
            remove-private-AS
            soft-reconfiguration inbound always
  !
 !
!
```

Important lines are:

1. `next-hop-self` - this is not default in this case!

## Testing

### Are prefixes received?

Example:

```
# show bgp ipv4 unicast neighbors 2001:db8:1::2 received routes

   Network            Next Hop            Metric LocPrf Weight Path
*  192.0.2.0/24      2001:db8:1::2
                                                             0 64496 i

```

### Is the next-hop address resolution working?

```
#sho ip route 192.0.2.0/24

Routing entry for 192.0.2.0/24
  Known via "bgp 64500", distance 20, metric 0
  Tag 60767, type external
  Installed Oct 18 07:02:53.061 for 2d05h
  Routing Descriptor Blocks
    fe80::3817:fcff:fea0:1f17, from 2001:db8:1::2, via GigEthernet 0/1/1, BGP external
      Nexthop in Vrf: "default", Table: "default", IPv6 Unicast, Table Id: 0xe0800000
      Route metric is 0
  No advertising protos.
```


