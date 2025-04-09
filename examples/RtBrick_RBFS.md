# RtBrick RBFS

## Minimum Software Version

All versions of RBFS support RFC5549.

## Configuration

```
interface ifp-0/0/0 {
  unit 0 {
    address {
      ipv6 2001:db8:0:1::/127 {
      }
    }
  }
}
interface lo-0/0/0 {
  unit 0 {
    address {
      ipv4 192.0.2.1/32 {
      }
      ipv6 2001:db8::1/128 {
      }
    }
  }
}
instance default {
  protocol {
    bgp {
      hostname B1;
      local-as 65501;
      router-id 192.0.2.1;
      address-family ipv4 unicast {
        redistribute direct {
        }
      }
      address-family ipv6 unicast {
        redistribute direct {
        }
      }
      peer {
        ipv6 2001:db8:0:1::1 2001:db8:0:1::0 {
          peer-group peer-65502;
        }
      }
      peer-group peer-65502 {
        remote-as 65502;
        address-family ipv4 unicast {
          extended-nexthop true;
          nexthop-self true;
        }
        address-family ipv6 unicast {
          nexthop-self true;
        }
      }
    }
  }
}

```

<details>

<summary>Configuration in 'set' format</summary>


```
  set interface ifp-0/0/0 
  set interface ifp-0/0/0 unit 0 
  set interface ifp-0/0/0 unit 0 address 
  set interface ifp-0/0/0 unit 0 address ipv6 2001:db8:0:1::/127 
  set interface lo-0/0/0 
  set interface lo-0/0/0 unit 0 
  set interface lo-0/0/0 unit 0 address 
  set interface lo-0/0/0 unit 0 address ipv4 192.0.2.1/32 
  set interface lo-0/0/0 unit 0 address ipv6 2001:db8::1/128 
  set instance default 
  set instance default protocol bgp hostname B1
  set instance default protocol bgp local-as 65501
  set instance default protocol bgp router-id 192.0.2.1
  set instance default protocol bgp address-family ipv4 unicast 
  set instance default protocol bgp address-family ipv4 unicast redistribute direct 
  set instance default protocol bgp address-family ipv6 unicast 
  set instance default protocol bgp address-family ipv6 unicast redistribute direct 
  set instance default protocol bgp peer ipv6 2001:db8:0:1::1 2001:db8:0:1::0 
  set instance default protocol bgp peer ipv6 2001:db8:0:1::1 2001:db8:0:1::0 peer-group peer-65502
  set instance default protocol bgp peer-group peer-65502 
  set instance default protocol bgp peer-group peer-65502 remote-as 65502
  set instance default protocol bgp peer-group peer-65502 address-family ipv4 unicast 
  set instance default protocol bgp peer-group peer-65502 address-family ipv4 unicast extended-nexthop true
  set instance default protocol bgp peer-group peer-65502 address-family ipv4 unicast nexthop-self true
  set instance default protocol bgp peer-group peer-65502 address-family ipv6 unicast 
  set instance default protocol bgp peer-group peer-65502 address-family ipv6 unicast nexthop-self true

```

</details>


Important lines are:

1. At least one (loopback) interface with an IPv4 address. This address will be used for answering ICMP unreachables or during traceroutes.(*)
2. `nexthop-self true` as this is not the default.
4. `extended-nexthop true` needs to be enabled to negotiate IPv4 NRLIs with IPv6 nexthop.


## Testing

### Are the routes getting accepted?

Example:

```
B2.rtbrick.net: op> show bgp rib-in ipv4
Flags: & - Imported, ! - Error, N - RPKI Unknown, I - RPKI Invalid, V - RPKI Valid
Instance: default, AFI: ipv4, SAFI: unicast
  Hostname: Local, Peer IP: 0.0.0.0 
  Source IP: 0.0.0.0, Total routes: 1
    Flags  Prefix                                       Next Hop                           MED         Lpref       AS Path
           192.0.2.2/32                                 -                                  0           100         -
  Hostname: B1, Peer IP: 2001:db8:0:1:: 
  Source IP: 2001:db8:0:1::1, Total routes: 2
    Flags  Prefix                                       Next Hop                           MED         Lpref       AS Path
           192.0.2.1/32                                 2001:db8:0:1::                     0           -           65501

```

### Is next-hop address resolution working?

Yes:

```
B2.rtbrick.net: op> show route prefix 192.0.2.1/32 detail
Instance: default, AFI: ipv4, SAFI: unicast
  Prefix: 192.0.2.1/32
    Source: bgp, Preference: 20
      Next Hop: 2001:db8:0:1::
      Covering prefix: 2001:db8:0:1::/128
      Next Hop type: direct, Next Hop action: None
      Resolved in: default-ipv6-unicast(Primary)
      Outgoing Interface: ifp-0/0/0/0, NextHop MAC: 7a:01:61:c0:00:00


```

For more information, see the [RBFS documentation](https://documents.rtbrick.com/)