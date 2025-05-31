# Bird2 enhanced next hop configuration

## Minimum Software Requirements

Bird2 has supported IPv6 next hops for IPv4 routes since version 2.0.8.

## Configuration

The key setting is **extended next hop on** in the IPv4 channel of the protocol:
```
protocol bgp EXAMPLE {
  neighbor 2001:db8:3::2 as 65533;
  ipv4 {
    import all;
    export all;
    extended next hop on;
  }
}
```
IPv6 GUA/LLA next-hop preference can be configured by setting `next hop prefer global` within a channel.

## Testing
### Are the routes getting accepted and installed?

Yes:
```
zsh% birdc show route protocol EXAMPLE
Table master4:
192.0.2.0/24  unicast [EXAMPLE 2024-12-12] * (100) [AS65533i]
  via 2001:db8:3::2 on eth0
zsh% ip route show 192.0.2.0/24
192.0.2.0/24 via inet6 2001:db8:3::2 dev eth0 proto bird metric 32
```

### Is the next-hop address resolution working?
yes

### What does traceroute look like?
The Linux router will use an IPv4 address to respond to traceroute in the following order:
 * if the outgoing interface (for the response) has an IPv4 address, use that
 * use any other interface has an IPv4 address, use that

### What happens if peer is not configured to use RFC8950 next hops?

**Not tested yet**

### Route filtering?

You can specify separate filters per channel (e.g. ipv4 or ipv6).
The config example above only has the ipv4 channel defined and
it has the "all" filter defined for both import and export.
