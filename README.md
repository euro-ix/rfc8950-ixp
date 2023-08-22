# rfc8950-ixp
Repository for the rfc8950-ixp working group

# RFC8950 Working Group Charter - Euro-IX

## Scope

The main purpose of this document is to describe the activities of a new working group on using IPv6 next-hops for IPv4 routes at IXPs. In particular, it will describe the purpose for which the working group is being set up, the activities that will be carried out, how they will be carried out and timing.

## Background and goal

Previously unused IPv4 address space has run out, and as IXPs grow, it’s now increasingly difficult to find a new bigger IPv4 subnet for the shared medium. One proposed solution that was put forward some years ago (RFC5549) was to use only IPv6 on the IXP shared medium and disassociate ARP from mapping route next-hops to MAC addresses on the IXP. The specification has recently been updated as RFC8950.
As vendors have finally started to implement this functionality, it’s time to figure out how to deploy it at IXPs. Most of the deployment happens at IXP members, but the IXP is involved at a few points such as the route servers and to rule on disputes in some corner cases.
[Give some background information on the activity, why a working group is needed and the working groups goal e.g. a document to explain best practises for this activity]


## Plan and timelines

The project planning is described below:

### Step 1 (2 weeks) - Creation of the working group

A call for volunteers will be sent to the Euro-IX mailing list. In the initial phase a group of five (5)[X] number of people needed for the working group. Over time and during the activities, the group may make further requests to the community for additional people. This will depend on the overall performance and the effort required to get the job done.


### Step 2 - First meeting of the working group (23 Apr 23)

The first meeting will be important to kick off the group's activities and it will have the following goals: 

 * presentation of participants 
 * chair and co-chair selection 
 * overview of activities 
 * discussion of working methods and tools (ML, Mattermost, Slack, etc.) 
 * roles in the project 
 * frequency of meetings 
 * timelines. 

The initial agenda of the meeting will be established in advance by the promoters.

### Step 3 (3 months) - First activities of the working group

[describe steps that the working group will be taking]
 * Definition of requirements to members
 * Evaluation of vendor support (routers, route-servers) [based on documentation]
 * Virtual test environment(?)
 * IXP Manager integration, identify gaps

At the end of the first 3 steps of the project, there will be a discussion to initiate the second phase if needed, i.e., the creation or adaptation of one of the identified solutions 


### Step 4 [If needed, will depend on the output of the first phase]. 

[If a step 4 is require, describe the detail here]
 * Interop-testing(?) [how can we connect real equipment?]
 * Create configuration guides for different router vendors
 * Identify implications to filtering
 * Can we support brownfield? Migration path? How to support/integrate legacy equipment?



## Suitable profiles for this working group

We would like to invite volunteers from Euro-IX (or other interested parties) to join us to do this [describe activity]. 

As we have some defined goals and targets, we would like to see some of the following skillsets join the working group:

[Describe skill set requirements for working group]
 * IXP Manager setup
 * IXP Manager “hacking”/customisation
 * Arouteserver configuration/customisation
 * BIRD2 route-server configuration (Linux)
 * OpenBGPd route-server configuration (OpenBSD, Linux)
 * Technical writing
 * Setup and operation of virtual environment
 * Knowledge about router vendor X – configuration, access to documentation, ideally access to equipment
