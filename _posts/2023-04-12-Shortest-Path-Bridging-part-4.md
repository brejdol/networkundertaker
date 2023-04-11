**Pseudo-wires**

Even more MPLS-lingo? Yeah, I guess. Pseudo-wires are well known in the industry, mainly for the price-point we all came to hate. Yes, I'm looking at you, ISP...person? We already created an L2VPN-service in the last post, what gives? Well, there is a difference - An L2VPN is one-to-many. A pseudo-wire is *one-to-one* ONLY. Even if it is possible, you must never enable a pseudo-wire service on more then exactly two switchports in your environment. Enabling the pseudo-wire function turns OFF all packet replication, and all source-learning. Because there is nothing to replicate to multiple destinations, and really nothing to learn. All frames that comes in on port A should be forwarded to port B, and all frames that comes in on port B should be forwarded to port A. Hence, the name, pseudo-wire. You should always use the default headend multicast replication, and you should always allow all on the service:

```
BEB1-> service access port 1/1/48
BEB1-> service 1000 spb isid 1000 bvlan 4001 pseudo-wire enable
BEB1-> service 1000 sap port 1/1/48:all
```

Pseudo-wires presents an interesting variation to a straight-ahead L2VPN - Since everything is forwarded as-is, and nothing is learned, vlan-separation will be honoured, and forwarded as-is, in contrast to the "normal" service, where one service equals one broadcast domain. This means you could for example choose pseudo-wires to connect "islands" of old legacy vlans running in older non-SPB-switches. The psedo-wires gives an isolated path through the SPB-backbone. Pseudo-wires also have the advantage of being extremely slim resource wise. Again, nothing is learned, nothing is remembered. Everything is just forwarded as-is between two ports. And the separation is of course total, so you can have as many pseudo-wires you want, and all of them can carry the same vlan-ids, but for different customers.
This setup has a lot of potential uses - Think about HA-traffic for example - Firewall clusters, routers, databases etc. Every two things that just need a cable between them, but unfortunatly they aren't in the same datacenter - All these can be solved with pseudo-wires. Older protocols that isn't really IP, but happen to use a networking cable - Perfect case for pseudo-wires. There's probably a lot more cases that I can't think of now. 

SPB is all about bridging, everything is bridged. So how do you route? Let's find out.

[Shortest Path Bridging Part 5 - Routing](https://networkundertaker.com/2023/04/12/Shortest-Path-Bridging-part-5.html)

[Start page]({{ '/' | absolute_url }})