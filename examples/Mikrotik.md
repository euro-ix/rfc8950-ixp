# Mikrotik RouterOS
Credit to Amin Younessi for the configuration.

## Known Issues
When running `/routing/bgp/instance/print`, the extended next-hop encoding capability ("`enhe`") seems to not be shown for the `local.capabilities` attribute, even though it is advertised to peers (can be checked on the receiving end). The `remote.capabilities` attribute is displayed correctly, though.

## Minimum Software Version
### Supported since
[7.20beta5](https://mikrotik.com/download/changelogs#c-testing-v7_20beta5)
### Tested with
7.20beta5

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
/interface ethernet
set ether1 comment=peering_lan_interface
set ether2 comment=client_lan_interface
/routing bgp instance
add as=65001 name=bgp-instance-1 router-id=203.0.113.1
/routing bgp template
set default afi=ip input.filter=default_accept_filter output.filter-chain=accept_client_lan_filter .network=client_lan_address_list
/ip address
add address=192.0.2.1/24 interface=ether2
add address=203.0.113.1/32 interface=lo
/ip firewall address-list
add address=192.0.2.0/24 list=client_lan_address_list
/ipv6 address
add address=3fff::1 interface=ether1
/routing bgp connection
add instance=bgp-instance-1 local.role=ebgp name=bgp-connection-1 remote.address=3fff::2 templates=default
/routing filter rule
add chain=accept_client_lan_filter rule="if ( dst == 192.0.2.0/24 ) { accept; }"
add chain=accept_client_lan_filter rule=reject
add chain=default_accept_filter rule=accept
```

### Important lines
There is no special configuration needed, neither for announcing the extended next-hop encoding capability nor for forwarding IPv4 packets on IPv6 interfaces.

## Testing
### Are the routes getting accepted?
Yes.

Example:
tba

### Is next-hop address resolution working?
Yes.

Example:
tba

### What does traceroute look like?
The router is visible with its loopback interface IPv4 address.

### What happens if peer is not configured to use RFC8950 next hops?
tba

### What happens if peer sets an IPv4 address on the peering interface?
tba