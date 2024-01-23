# GoBGP

GoBGP supports extended nexthops natively. GoBGP relies on third party integrations to interact with the FIB, e.g. FRR. So any forwarding plane functionality is out-of-scope for the test.

## Minimum Software Version

GoBGP seems to support the extended nexthop capability for a while. I could not identify any "first version".

## Configuration

No special configuration is necessary. Just the address family ipv4-unicast needs to be enabled for the neibhbor.

## Testing

GoBGP will announce and negotiate extended next-hop capability with the peer.

### Are the routes getting accepted?

Yes:

```
bash-5.0# gobgp global rib
   Network              Next Hop             AS_PATH              Age        Attrs
*> 10.1.0.0/24          2001:7f8:19:6::1:1   65001                00:16:39   [{Origin: ?} {Communities: 64512:10, 64512:21, 64512:41} {Extcomms: [64512:10], [64512:21], [64512:41]} {LargeCommunity: [ 16374:64512:10, 16374:64512:21, 16374:64512:41]} {Flags: TRANSITIVE|OPTIONAL, Type: BGPAttrType(35), Value: ?�}]
*> 10.1.1.0/24          2001:7f8:19:6::1:1   65001                00:16:39   [{Origin: ?} {Communities: 64512:10, 64512:21, 64512:41} {Extcomms: [64512:10], [64512:21], [64512:41]} {LargeCommunity: [ 16374:64512:10, 16374:64512:21, 16374:64512:41]} {Flags: TRANSITIVE|OPTIONAL, Type: BGPAttrType(35), Value: ?�}]
*> 10.7.0.0/24          2001:7f8:19:6::7:1   65007                00:57:37   [{Origin: i} {Med: 100} {LocalPref: 100} {Communities: 1:1, 2:2}]
```

### Is the next-hop address resolution working?

No FIB integration tested.

### What does traceroute look like?

No FIB integration tested.

### What happens if peer is not configured to use RFC8950 next hops?

No FIB integration tested.
