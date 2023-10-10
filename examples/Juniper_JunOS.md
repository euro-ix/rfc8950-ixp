# Juniper JunOS

## Minimum Software Version

The minimum JunOS version to support RFC5549 is 17.3R1. This version introduced the `extended-nexthop` term.

TODO: The [documentation](https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/multiprotocol-bgp.html#id-understanding-redistribution-of-ipv4-routes-with-ipv6-next-hop-into-bgp) talks about `dynamic-tunnel`. We do not need this, but it remains to be determined which JunOS version supports untunneled IPv4 with IPv6 next-hop.

## Configuration

```
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet;
            family inet6 {
                address 2001:7f8:19:6::11:1/64;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.1.11.255/32; 
            }
        }
    }
}
policy-options {
    policy-statement nh-self {
        then {
            next-hop self;
        }
    }
}
routing-options {
    router-id 10.0.0.11;
    autonomous-system 65011;
}
protocols {
    bgp {
        group peer-a {
            type external;
            local-address 2001:7f8:19:6::11:1;
            family inet {
                unicast {
                    extended-nexthop;
                }
            }
            family inet6 {
                unicast;
            }
            export nh-self;
            peer-as 65001;
            local-as 65011;
            neighbor 2001:7f8:19:6::1:1;
        }
    }
}
```

There are four important lines:

1. `interfaces X unit Y family inet` needs to be enabled to allow IPv4 forwarding on an interface.
2. At least one (loopback) interface with an IPv4 address. This address will be used for answering ICMP unreachables or during traceroutes.
3. `policy-statement nh-self then next-hop self` should be default for IXP connection, but it's even more important here.
4. `protocols bgp group X family inet unicast extended-nexthop` needs to be enabled to negotiate the capability.

## Testing

### Are the routes getting accepted?

Example:

```
> show route receive-protocol bgp 2001:7f8:19:6::1:1 

  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.1.0.0/24             2001:7f8:19:6::1:1                      65001 ?
* 10.1.1.0/24             2001:7f8:19:6::1:1                      65001 ?
* 10.1.2.0/24             2001:7f8:19:6::1:1                      65001 65002 ?
* 10.1.3.0/24             2001:7f8:19:6::1:1                      65001 65002 ?

> show route 10.1.2.0 detail 

10.1.2.0/24 (1 entry, 1 announced)
        *BGP    Preference: 170/-101
                Next hop type: Router, Next hop index: 660
                Address: 0x77c62a4
                Next-hop reference count: 12, Next-hop session id: 322
                Kernel Table Id: 0
                Source: 2001:7f8:19:6::1:1
                Next hop: 2001:7f8:19:6::1:1 via ge-0/0/0.0, selected
                Session Id: 322
                State: <Active Ext>
                Local AS: 65011 Peer AS: 65001
                Age: 19:58 
                Validation State: unverified 
                Task: BGP_65001_65011.2001:7f8:19:6::1:1
                Announcement bits (3): 0-KRT 2-BGP_RT_Background 3-Resolve tree 1 
                AS path: 65001 65002 ? 
                Accepted
                Localpref: 100
                Router ID: 10.0.0.1
                Thread: junos-main 
```

### Is next-hop address resolution working?

Yes:

```
> show route 10.1.2.0
10.1.2.0/24        *[BGP/170] 00:27:21, localpref 100
                      AS path: 65001 65002 ?, validation-state: unverified
                    >  to 2001:7f8:19:6::1:1 via ge-0/0/0.0

> show ipv6 neighbors 
IPv6 Address                            Linklayer Address  State       Exp   Rtr  Secure  Interface
2001:7f8:19:6::1:1                       0c:00:3e:ef:d1:a2  reachable   24    yes  no      ge-0/0/0.0
```

### What does traceroute look like?

The router is visible with its loopback interface IPv4 address.

### What happens if peer is not configured to use RFC8950 next hops?

If extended next-hop cannot be negotiated, neither IPv4 routes can be received not advertised. Status can be checked:

```
> show bgp neighbor 2001:7f8:19:6::1:1 | match nexthop 
  NLRI that we support extended nexthop encoding for: inet-unicast
  NLRI that peer supports extended nexthop encoding for: inet-unicast
  NLRI(s) enabled for color nexthop resolution: inet-unicast inet6-unicast
```

In case `family inet` is not enabled on the interface connecting the IPv6 next-hop, the routes are being accepted and visible in the FIB, but not forwarded, i.e. blackholed.
