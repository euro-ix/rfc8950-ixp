# Config Examples

Put config examples for different platforms and routing daemons in this directory.

## Check List for Testing

### Configuration
 * Find out minimum software requirements
 * Configure EBGP peer that uses RFC8950 style next hops
 * Configure source address/interface for ICMP Unreachables, if possible
### Testing
 * What does traceroute look like?
 * What happens if peer is not configured to use RFC8950 next hops?
   - Errors? Warnings? Alerts?
   - Blackholing?
   - Turn down session as a fail safe?
