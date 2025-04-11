# Nokia SR-Linux

## Minimum Software Version

RFC8950 compliant functionalities have successfully been tested with Nokia SR Linux 22.6.4.

## Configuration

```
routing-policy {
    prefix-set Clients {
        prefix 10.0.0.0/24 mask-length-range exact {
        }
    }
    policy AcceptAny {
        default-action {
            accept {
            }
        }
    }
    policy Announce-Clients {
        statement 10 {
            match {
                prefix-set Clients
            }
            action {
                accept {
                }
            }
        }
    }
}
interface ethernet-1/1 {
    admin-state enable
    subinterface 0 {
        description "to IXP"
        admin-state enable
        ipv6 {
            address 2001:db8::1/64 {
            }
        }
    }
}

network-instance default {
    ip-forwarding {
        receive-ipv4-check false
    }
    interface ethernet-1/1.0 {
    }
    protocols {
        bgp {
            admin-state enable
            autonomous-system 65001
            router-id 10.0.0.1
            group rs {
                export-policy Announce-Clients
                import-policy AcceptAny
                peer-as 65254
                ipv4-unicast {
                    advertise-ipv6-next-hops true
                    receive-ipv6-next-hops true
                    admin-state enable
                }
            }
            neighbor 2001:db8::254 {
                peer-group rs
            }
        }
    }
}
```

There are three important lines:

1. `receive-ipv4-check false` in the context of ip-forwarding config. This allows to accept IPv4 packets even though the interface is operationally down for IPv4.
2. `receive-ipv6-next-hops true` in the bgp context. This enables accepting IPv4 routes with IPv6 next hops and announces the capability.
3. `advertise-ipv6-next-hops true` in the bgp context. This enables sending of IPv6 next hops for IPv4 routes.

## Testing

### Are the routes getting accepted?

Example:

```
--{ running }--[  ]--
A:srlinux# show network-instance default route-table ipv4-unicast summary
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
|                      Prefix                       |  ID   | Route Type |     Route Owner      |        Active        |  Metric  |  Pref   |        Next-hop (Type)        |      Next-hop Interface       |
+===================================================+=======+============+======================+======================+==========+=========+===============================+===============================+
| 192.0.2.0/24                                      | 3     | local      | net_inst_mgr         | True                 | 0        | 0       | 192.0.2.1 (direct)            | ethernet-1/2.0                |
| 192.0.2.1/32                                      | 3     | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)                | None                          |
| 192.0.2.255/32                                    | 3     | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast)              |                               |
| 203.0.113.0/24                                    | 0     | bgp        | bgp_mgr              | True                 | 0        | 170     | 2001:db8::222 (indirect)      | ethernet-1/1.0                |
+---------------------------------------------------+-------+------------+----------------------+----------------------+----------+---------+-------------------------------+-------------------------------+
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv4 prefixes with active routes     : 4
IPv4 prefixes with active ECMP routes: 0
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
A:srlinux#
```

### Is next-hop address resolution working?

Yes:

```
--{ running }--[  ]--
A:srlinux# show network-instance default protocols bgp routes ipv4 prefix 203.0.113.0/24
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------Show report for the BGP routes to network "203.0.113.0/24" network-instance  "default"
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------Network: 203.0.113.0/24
Received Paths: 1
  Path 1: <Best,Valid,Used,>
    Route source    : neighbor 2001:db8::254
    Route Preference: MED is -, LocalPref is 100
    BGP next-hop    : 2001:db8::222
    Path            :  i [65222]
    Communities     : None
Path 1 was advertised to:
[  ]
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------{ running }--[  ]--
A:srlinux# show network-instance default protocols bgp neighbor 2001:db8::254
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------BGP neighbor summary for network-instance "default"
Flags: S static, D dynamic, L discovered by LLDP, B BFD enabled, - disabled, * slow
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------+------------------------------------------+----------------------------+----------+---------------+-----------------------+-----------------------+--------------------+------------------------------------------+
|          Net-Inst          |                   Peer                   |           Group            |  Flags   |    Peer-AS    |         State         |        Uptime         |      AFI/SAFI      |              [Rx/Active/Tx]              |
+============================+==========================================+============================+==========+===============+=======================+=======================+====================+==========================================+
| default                    | 2001:db8::254                            | rs                         | S        | 65254         | established           | 0d:0h:12m:43s         | ipv4-unicast       | [1/1/1]                                  |
+----------------------------+------------------------------------------+----------------------------+----------+---------------+-----------------------+-----------------------+--------------------+------------------------------------------+
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------Summary:
1 configured neighbors, 1 configured sessions are established,0 disabled peers
0 dynamic peers
--{ running }--[  ]--
A:srlinux#
```

### What does traceroute look like?
Trace is from client connected to ethernet-1/2:
```
[root@client1 /]# traceroute 203.0.113.1
traceroute to 203.0.113.1 (203.0.113.1), 30 hops max, 60 byte packets
 1  192.0.2.1 (192.0.2.1)  1.968 ms * *
 2  203.0.113.1 (203.0.113.1)  0.505 ms  0.632 ms  0.734 ms
[root@client1 /]#
```
a bigger setup would be interesting.

### What happens if peer is not configured to use RFC8950 next hops?
To be tested.

### How do you add route filters for IPv4?
To be tested.

### What happens if peer sets an IPv4 address on the peering interface?
To be tested.
