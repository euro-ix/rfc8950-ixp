# rfc8950-ixp
Repository for the rfc8950-ixp working group

# RFC8950 Working Group Charter - Euro-IX

## Scope

The main purpose of this document is to describe the activities of a new working group on using IPv6 next-hops for IPv4 routes at IXPs. In particular, it will describe the purpose for which the working group is being set up, the activities that will be carried out, how they will be carried out and timing.

## Background

Previously unused IPv4 address space has run out, and as IXPs grow, it’s now increasingly difficult to find a new bigger IPv4 subnet for the shared medium. One proposed solution that was put forward some years ago (RFC5549) was to use only IPv6 on the IXP shared medium and disassociate ARP from mapping route next-hops to MAC addresses on the IXP. The specification has recently been updated as RFC8950.
As vendors have finally started to implement this functionality, it’s time to figure out how to deploy it at IXPs. Most of the deployment happens at IXP members, but the IXP is involved at a few points such as the route servers and to rule on disputes in some corner cases.

## Goals and Targets

Define goals and targets of the working group:

 * Raise awareness for RFC8950 for interconnection
 * Raise awareness for interoperability issues
 * Make vendors aware of bugs / improvement ideas
 * Create recommendations for operators
 * Migrate IXP related work to [Euro-IX route server WG](https://github.com/euro-ix/route-server-wg)


### Skillset

 * IXP Manager setup
 * IXP Manager “hacking”/customisation
 * Arouteserver configuration/customisation
 * BIRD2 route-server configuration (Linux)
 * OpenBGPd route-server configuration (OpenBSD, Linux)
 * Technical writing
 * Setup and operation of virtual environment
 * Knowledge about router vendor X – configuration, access to documentation, ideally access to equipment

## Plan and timelines

The project planning is described below:

### Next steps

Continue working on the following topics:

 * Interop-testing (maybe extend how can we connect real equipment?)
 * Create configuration guides for different router vendors
 * Identify implications to filtering
 * Can we support brownfield? Migration path? How to support/integrate legacy equipment?

## Suitable profiles for this working group

We would like to invite volunteers from Euro-IX (or other interested parties) to join us to do this. 
