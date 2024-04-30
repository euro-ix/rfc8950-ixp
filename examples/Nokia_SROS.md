# Juniper JunOS

## Minimum Software Version

RFC8950 compliant functionalities have successfully been tested with Nokia SR OS 20.2.R1. 

## Configuration

```
    policy-options {
        prefix-list "Match1" {
            prefix 1.1.1.1/32 type exact {
            }
        }
        policy-statement "AcceptAny" {
            default-action {
                action-type accept
            }
        }
        policy-statement "Announce-Lo" {
            entry 10 {
                from {
                    prefix-list ["Match1"]
                }
                action {
                    action-type accept
                }
            }
        }
    }
    port 1/1/c1 {
        admin-state enable
        description "-"
        connector {
            breakout c1-100g
        }
    }
    port 1/1/c1/1 {
        admin-state enable
        description "to IXP"
        ethernet {
            mode network
        }
    }
    router "Base" {
        autonomous-system 65009
        interface "T1" {
            admin-state enable
            loopback
            ipv4 {
                primary {
                    address 10.9.0.1
                    prefix-length 32
                }
            }
        }
        interface "system" {
            admin-state enable
            ipv4 {
                primary {
                    address 10.99.99.1
                    prefix-length 32
                }
            }
            ipv6 {
                address 2001:db8::9 {
                    prefix-length 128
                }
            }
        }
        interface "to IXP" {
            admin-state enable
            port 1/1/c1/1
            ipv6 {
                forward-ipv4-packets true
                address 2001:7f8:19:6::9:1 {
                    prefix-length 64
                }
            }
        }
        bgp {
            admin-state enable
            family {
                ipv4 true
            }
            import {
                policy ["AcceptAny"]
            }
            export {
                policy ["Announce-Lo"]
            }
            extended-nh-encoding {
                ipv4 true
            }
            advertise-ipv6-next-hops {
                ipv4 true
            }
            group "4over6" {
            }
            neighbor "2001:7f8:19:6::1:1" {
                admin-state enable
                group "4over6"
                peer-as 65001
                import {
                    policy ["AcceptAny"]
                }
                export {
                    policy ["Announce-Lo"]
                }
            }
        }
    }
```

There are three important lines:

1. `forward-ipv4-packets true` in the context of the interface config. This allows to forward IPv4 packets even though the interface is operationally down for IPv4.
2. `extended-nh-encoding { ipv4 true }` in the bgp context. This enables accepting IPv4 routes with IPv6 next hops.
3. `advertise-ipv6-next-hops { ipv4 true }` in the bgp context. This enables sending of IPv6 next hops for IPv4 routes.

## Testing

### Are the routes getting accepted?

Example:

```
A:admin@peer-t# show router route-table

===============================================================================
Route Table (Router: Base)
===============================================================================
Dest Prefix[Flags]                            Type    Proto     Age        Pref
      Next Hop[Interface Name]                                    Metric   
-------------------------------------------------------------------------------
10.1.0.0/24                                   Remote  BGP       00h01m10s  170
       2001:7f8:19:6::1:1                                           0
10.1.1.0/24                                   Remote  BGP       00h01m10s  170
       2001:7f8:19:6::1:1                                           0
10.9.0.1/32                                   Local   Local     01h12m56s  0
       T1                                                           0
10.99.99.1/32                                 Local   Local     04d23h46m  0
       system                                                       0
-------------------------------------------------------------------------------
No. of Routes: 4
Flags: n = Number of times nexthop is repeated
       B = BGP backup route available
       L = LFA nexthop available
       S = Sticky ECMP requested
===============================================================================
```

### Is next-hop address resolution working?

Yes:

```
A:admin@peer-t# routes 10.1.1.0
===============================================================================
 BGP Router ID:10.99.99.1       AS:65009       Local AS:65009      
===============================================================================
 Legend -
 Status codes  : u - used, s - suppressed, h - history, d - decayed, * - valid
                 l - leaked, x - stale, > - best, b - backup, p - purge
 Origin codes  : i - IGP, e - EGP, ? - incomplete

===============================================================================
BGP IPv4 Routes
===============================================================================
Flag  Network                                            LocalPref   MED
      Nexthop (Router)                                   Path-Id     IGP Cost
      As-Path                                                        Label
-------------------------------------------------------------------------------
u*>?  10.1.1.0/24                                        None        None
      2001:7f8:19:6::1:1                                 None        0
      65001                                                          -
-------------------------------------------------------------------------------
Routes : 1
===============================================================================

A:admin@peer-t# show router neighbor

===============================================================================
Neighbor Table (Router: Base)
===============================================================================
IPv6 Address                                   Interface                  
   MAC Address                State         Expiry          Type         RTR
-------------------------------------------------------------------------------
2001:7f8:19:6::1:1                             to IXP
   0c:00:4f:a6:28:45          STALE         03h59m59s       Dynamic      Yes
fe80::e00:4fff:fea6:2845                       to IXP
   0c:00:4f:a6:28:45          STALE         02h50m28s       Dynamic      No
-------------------------------------------------------------------------------
No. of Neighbor Entries: 2
===============================================================================
```

### What does traceroute look like?
To be tested.
