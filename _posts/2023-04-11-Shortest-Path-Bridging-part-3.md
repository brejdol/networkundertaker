**Creating Services - L2VPNs**

A service in SPB is what you want bridged from A to B in your network. A service is really a broadcast domain within the SPB framework (regarding source-learning, flooding etc), so it works very much like a legacy VLAN. As stated before, even L3 services are in fact bridged. The main operational difference is that a VLAN has to be configured in all switches in the path between ingress/egress ports. A service only needs to be configured in the edge nodes. If there is 100 switches in between is totally irrelevant. This is because a node that has a service will advertise in the control BVLAN that it has that service to all other nodes. All nodes that have the unique service isid and have access ports configured will be bridged together, effectively creating an L2VPN between them. All other nodes that doesn't have that isid configured doesn't care. They just switch the traffic.

*A node in the SPB domain will always be root for the incoming service mac-addresses it forwards into the SPB domain.* 

This is one of the ways the SPB control plane makes sure everything is ok - If the same mac shows up in another switch even though it is bound to a service somewhere else, that traffic will be dropped. Mac moves between switches, for example when a virtual machine moves to another host isn't an issue, since that mac doesn't really exist in two switches at the same time.

A service needs four settings to go online:
* A service number between 1-16777216. This service number only locally significant, but please, use the same everywhere!
* An isid number between 256-16777216. This number is globally unique in your SPB domain. And yes, the first 255 is reserved and can not be used, so if you have VLANS under 256, and want to create services that map to them (which is handy and easy to remember), start with 10xxx or something. you have a lot of service/isids to choose from...
* You need to map your service to a bvlan, because the service will be encapsulated with an PBB-header within that BVLAN.
* You need to map your service to a Service Access Port (SAP), which is the customer-facing port, where the PBB-header is stripped off, and the payload is delivered.

```
BEB1-> service access port 1/1/48
BEB1-> service 1000 spb isid 1000 bvlan 4001
BEB1-> service 1000 sap port 1/1/48:0
```
Pretty self-explanatory, no? We choose a physical port to be the service access port, i.e the customer facing port, create the service 1000 with the isid 1000 and map this to our traffic BVLAN 4001 (BVLAN 4000 was dedicated to the control plane traffic, remember?). The last line states that the service should be sent out on SAP port 1/1/48 untagged.
The last line can also look like this:

```
BEB1-> service 1000 sap port 1/1/48:1000
```

This means service 1000 is sent out on SAP port 1/1/48 tagged with the VLAN-id 1000. But it doesn't end there, we can also do this:

```
BEB1-> service 1000 sap port 1/1/48:1000:3500
```

This means we double-tagged the outgoing traffic with 1000 as the outer s-vlan-id, and 3500 as the inner c-vlan. Effectively using Q-in-Q, or VLAN stacking if you will. You can actually stack one more, doing triple tagging. But wait, there is more:

```
BEB1-> service 1000 sap port 1/1/48:all
```
All? Yep. This means the SAP port will accept any untagged or tagged, or double/triple-tagged frame. Everything is accepted. You can send untagged, tagged, and double-tagged traffic at the same time, but do remember that a standard service is the equivalent to a broadcast domain, so you might want to keep the 1-to-1 mapping service-to-vlan. 1 vlan = 1 service IS the best practice. The service name and ISID have no correlation at all to the settings you choose on your SAP port. Nothing! That takes a while to get used to.The above examples does however assume all sides of the service are the same. Usually, you would want a bit more flexibility than that. And you can get that:

```
BEB1-> service access port 1/1/48 vlan-xlation enable
BEB1-> service 1000 spb isid 1000 bvlan 4001 vlan-translation enable
BEB1-> service 1000 sap port 1/1/48:1000 
```

This setting has to be done both on the SAP port and on the service itself, on all switches that have the service, and it means you can do whatever you want with the service - You can have it tagged in one switch, as above, untagged on another, you can use different tags everywhere if you want to. The service is what it is, and you gets to choose what you want it to be on the SAP port. 

*There is no correlation between the service and what you choose to ingress/egress with.* 

Wrap that around your head for a bit. If you come from a VLAN-based environment, this is seriously weird. On the other hand, if you come from a service provider background where you have MPLS/SR in the core, this isn't weird at all - Since there you also have a transparent core that doesn't care much about what it switches, it is at the edges you decide what will happen.
Let's do a summary:

* A service is a broadcast domain (there are exceptions to this, we will go into that in the next chapter).
* A service is switched transparently in the SPB domain. Nodes that doesn't have edge (SAP) ports enabled doesn't care, they just switch what's in the BVLAN.
* You can choose how you present the service on client facing ports (SAPs).

The service itself also have more settings that can be of interest:

* You can set the multicast replication mode - We'll cover that further down
* By default, a PBB frame include all original tags, but you can remove tags on the ingress port: remove-ingress-tag enable
* You can also map an L2-profile to the service, where you can choose to tunnel/drop STP-bpdus etc

The big thing with a service is that you ONLY have to create it on the ingress/egress SPB-nodes that will serve clients. When you create a service, its ISID will be advertised out to ALL nodes that have a service with the same ISID, effectively creating an L2VPN between these nodes, and these nodes only. All other nodes just switch the BVLAN, and think nothing more of it. As you see, this is very much like creating a VLAN, and stretching it between switches. But with a service, all this work is done by the IS-IS underlay and the BVLAN, your service is advertised via the control plane. It doesn't matter if you have 100 switches inbetween the nodes you enable the service on - If they have an SPB adjacency, they will have this L2VPN service between them, spanning over the redundant SPB-mesh backbone.

**BUM-traffic**

SPB have three different BUM (broadcast, unknown unicast, multicast) modes:

*Head-end*: BUM traffic that is recieved on a SAP port is replicated on the ingress BEB-node and converted to multiple unicast frames. A replica is created for every other BEB that has the same isid and is forwarded using the unicast FDB.
This can consume quite a bit of bandwidth, but is easy on the control plane.

*Tandem (S,G): This mode use a separate multicast shortest path tree and forwarding database. Tandem replication uses very little bandwidth, but use more of the control plane resources due to that it addes a separate SPT/FDB per service. In a small environment, this doesn't matter.

*Tandem (*,G)*: This mode uses a separate multicast tree. This is not a SPT, and is not confruent with the unicast SPT. A multicast (*,G) is created for every BVLAN using Tandem (*,G) multicast replication. This tree is similar to a STP, and is rooted at one SPB node according to the bridge priority. This mode is a compromise between bandwidth and control plance resources. This might be the best option if most traffic is sourced or destined towards the root bridge.

I usually use Tandem(S,G) mode, since I only handle small environments. Control plane load isn't an issue.

**Convergence**

So what happens when a link fails? Well, it's the same as always - A clean failure where a fiber is cut, an SFP dies, or a switch in the path loses power is easy to handle. SPB has very very fast convergence times, usually under 50 ms if your network isn't huge. As you add more nodes, the convergence time will be slower, but you need a sizable network to actually notice that. I would say you need to have way over 100 nodes to actually notice it. I only have access to smaller SPB meshes with around 20 nodes, and I can't really measure the outage if I pull a fiber, or pull the power on a switch in the path. It isn't really noticable, even if you are in a video call. With this said - Failures are often not that clean. An SFP can for example fail with a brown-out, not totally down, but working, then throwing errors and dropping traffic when warm etc etc. Those kinds of failures can still be messy. SPB is almost magic. Tragically enough, the ol' brown-outs still happen, even with SPB.

Next up: More types of services. How about... Pseudo-wires?

[Shortest Path Bridging Part 4 - Creating Services - Pseudo-wires](https://networkundertaker.com/2023/04/12/Shortest-Path-Bridging-part-4.html)

[Start page]({{ '/' | absolute_url }})
