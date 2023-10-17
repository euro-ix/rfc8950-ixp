# Config Examples

Put config examples for different platforms and routing daemons in this directory.

## Check List for Testing

### Configuration
 * Find out minimum software requirements
 * What's the configuration difference between IPv4 next-hops and RFC8950 next-hops over an IPv6 BGP session?
 * Configure EBGP peer that uses RFC8950 style next hops
 * Configure source address/interface for ICMP Unreachables, if possible
### Testing
 * Are the routes getting accepted?
 * Is next-hop address resolution working?
 * What does traceroute look like?
 * What happens if peer is not configured to use RFC8950 next hops?
   - Errors? Warnings? Alerts?
   - Blackholing?
   - Turn down session as a fail safe?
   - Other misconfiguration cases?
 * What happens if peer sets an IPv4 address on the peering interface?
   - Will the next-hops still be IPv6?
   - Will traceroutes now use this address instead of a loopback?
   - Could ISPs set up a /32 address on the interface to have a specific name show up in traceroutes?
