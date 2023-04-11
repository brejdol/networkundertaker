**Creating Services - L2VPNs**

A service in SPB is what you want transported from A to B in your network. A service is handled like a broadcast domain within the SPB framework (regarding source-learning, flooding etc), so it works very much like a legacy VLAN. As stated before, even L3 services are in fact bridged.
A service needs four settings to go online:
* A service number between 1-16777216. This number only locally significant, but please, use the same everywhere!
* An isid number between 256-16777216. This number needs to be globally unique in your SPB domain. And yes, the first 255 can not be used, so if you have VLANS under 256, and want to create services that map to them (wich is handy and easy to remember), start with 10xxx or something. you have a lot of service/isids to choose from...
* You need to map your service to a bvlan, because the service will be encapsulated with an PBB-header within that BVLAN.
* You need to map your service to a Service Access Port (SAP=, wich is the customer-facing port, where the PBB-header is stripped off, and the payload is delivered.)

```
BEB1-> service access port 1/1/48
BEB1-> service 1000 spb isid 1000 bvlan 4001
BEB1-> service 1000 sap port 1/1/48:0
```
Pretty self-explanatory, no? We choose a åhysical port to be SAP, create the service 1000 with the isid 1000 and map this to our traffic BVLAN 4001 (BVLAN 4000 was dedicated to the control plane traffic, remember?). The last line states that the service should be sent out on SAP port 1/1/48 untagged.
The last line can also look like this:

```
BEB1-> service 1000 sap port 1/1/48:1000
```

That means service 1000 is sent out on SAP port 1/1/48 tagged with the VLAN-id 1000. But it doesn't end there, we can do this:

```
BEB1-> service 1000 sap port 1/1/48:1000:3500
```

Hmm? This means we double-tagged the outgoing traffic with 1000 as the outer vlan-id, and 3500 as the inner. But wait, there is more:

```
BEB1-> service 1000 sap port 1/1/48:all
```
All? Yep. This means the SAP port will accept any untagged or tagged, or double-tagged frame. Everything is accepted. You can send untagged, tagged, and double-tagged traffic at the same time, but do remember that a standard service is the equivalent to a broadcast domain, so you might want to keep the 1-to-1 mapping, 1 vlan = 1 service IS best practice. The service name and ISID have no correlation at all to the settings you choose on your SAP port. Nothing! The above examples does however assume all sides of the service are the same. Usually, you would want a bit more flexibility than that. And you can get that:

```
BEB1-> service access port 1/1/48 vlan-xlation enable
BEB1-> service 1000 spb isid 1000 bvlan 4001 vlan-translation enable
BEB1-> service 1000 sap port 1/1/48:1000 
```

This setting has to be done both on the SAP port and on the service itself, and means you can do whatever you want with the service - You can have it tagged in one switch, as above, untagged on another, you can use different tags everywhere if you want to (I'm sure there is a user case for that somewhere). The service is what it is, and you gets to choose what you want it to be on the SAP port. The service itself also have more settings that can be of interest:

* You can set the multicast replication mode - Default is "headend", you can also choose "tandem".
* By default, a PBB frame include all original tags, but you can remove tags on the ingress port: remove-ingress-tag enable
* You can also map an L2-profile to the service, where you can choose to tunnel/drop STP-bpdus etc

The big thing with a service is that you ONLY have to create it on the ingress/egress SPB-nodes that will serve clients. When you create a service, its ISID will be advertised out to ALL nodes that have a service with the same ISID, effectively creating an L2VPN between these nodes, and these nodes only. As you see, this is very much like creating a VLAN, and stretching it between switches. But with a service, all this work is done by the IS-IS underlay and the BVLAN, your service is advertised via the control plane. It doesn't matter if you have 100 switches between the two nodes you enable the service on - If they have an SPB adjacency, they will have this L2VPN between them, spanning over the redundant SPB-mesh backbone.

**Convergence**

So what happens when a link fails? Well, it's the same as always - A clean failure where a fiber is cut, an SFP dies, or a switch in the path loses power is easy to handle. SPB has very very fast convergence times, usually under 50 ms if your network isn't huge. As you add more nodes, the convergence time will be slower, but you need a sizable network to actually notice that. I would say you need to have way over 100 nodes to actually notice it. I only have access to smaller SPB meshes with around 20 nodes, and I can't really measure the outage if I pull a fiber, or pull the power on a switch in the path. It isn't really noticable, even if you are in a video call. With this said - Failures are often not that clean. An SFP can for example fail with a brown-out, not totally down, but working, then throwing errors and dropping traffic when warm etc etc. Those kinds of failures can still be messy.

Next is: More types of services. How about... Pseudo-wires?


