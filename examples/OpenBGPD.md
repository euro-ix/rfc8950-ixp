# OpenBGPD

OpenBGPD supports RFC8950, but only installs routes for IPv4 prefixes with a IPv6 next-hop into the RIB, not the FIB ([at least yet](https://github.com/openbgpd-portable/openbgpd-openbsd/commit/f9365c08c05927510280d3b077392615c5b96026)).  
This is probably due to a limitation of *BSD based operating systems, as Linux supports this since version 5.2.

## Minimum Software Version
### Supported since
[8.8](https://www.openbsd.org/plus77.html)  
### Tested with
8.8

## Configuration
### Setup
#### Local
   - ASN: 6500**1**
   - Router ID: 203.0.113.**1**
   - Peering IP: 3fff::**1**
   - Network: 192.0.2.0/24
#### Remote
   - ASN: 6500**2**
   - Router ID: 203.0.113.**2**
   - Peering IP: 3fff::**2**
   - Network: 198.51.100.0/24

### Snippet
```
AS 65001
router-id 203.0.113.1

network 192.0.2.0/24

neighbor 3fff::2 {
   remote-as 65002

   announce IPv4 unicast
   announce extended nexthop yes
}

allow from any
allow to any
```

### Important lines
1. `announce extended nexthop yes`  
From [bgpd.conf(5)](https://man.openbsd.org/bgpd.conf.5):  
**announce extended nexthop (yes|no|enforce)**  
If set to **yes**, the extended nexthop encoding capability is announced. If negotiated, **IPv4 unicast** and **vpn** sessions can send paths with a IPv6 nexthop. If **enforce** is set, the session will only be established if the neighbor also announces the capability. The default is **no**.

## Testing

tba