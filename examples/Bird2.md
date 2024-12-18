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

## Testing
### Are the routes getting accepted and installed?

Yes:
```
zsh% birdc show route protocol EXAMPLE
Table master4:
192.0.2.0/24  unicast [EXAMPLE 2024-12-12] * (100) [AS65533i]
  via 2001:db8:3::2 on eth0
zsh% ip -4 route show 
```
unfinished

### Is the next-hop address resolution working?


### What does traceroute look like?


### What happens if peer is not configured to use RFC8950 next hops?

**Not tested yet**
