**Pseudo-wires**

Even more MPLS-lingo? Yeah, I guess. Pseudo-wires are well known in the industry, mainly for the price-point we all came to hate. Yes, I'm looking at you *\<insert ISP name\>!* 

We already created a L2VPN-service in the last post, what gives? Well, there is a difference - A L2VPN is one-to-many. A pseudo-wire is *one-to-one* ONLY. Even if it is possible (nothing will stop you), you must never enable a pseudo-wire service on more then *exactly* two ports/LAGs in your environment. Enabling the pseudo-wire function turns OFF all packet replication and all source-learning. There is simply nothing to replicate to multiple destinations, and really nothing to learn. All frames that comes in on port A should be forwarded to port B, and all frames that comes in on port B should be forwarded to port A. Hence the name, *pseudo-wire*. You should always use the default head-end multicast replication, and you should always allow all on the service SAPs. And since everything regarding replication is turned off, enabling a pseudo-wire service on more then *exactly* two ports will just create an unreliable service. Packets will get lost. Don't do it.

```
BEB1-> service access port 1/1/48
BEB1-> service 1000 spb isid 1000 bvlan 4001 pseudo-wire enable
BEB1-> service 1000 sap port 1/1/48:all
```

Pseudo-wires presents an interesting variation to a straight-ahead L2VPN services - Since everything is forwarded as-is and nothing is learned, vlan-separation will be honored and forwarded as-is, in contrast to the "normal" service, where one service equals one broadcast domain. This means you could for example choose pseudo-wires to connect "islands" of old legacy vlans running in older non-SPB-switches. The psedo-wires gives an isolated path through the SPB-backbone. 

Pseudo-wires also have the advantage of being extremely slim resource-wise for the CP. Again, nothing is learned, nothing is remembered. Everything is just forwarded as-is between two ports. And the separation is of course total, so you can have as many pseudo-wires you want, and all of them can carry the same VLAN-ids, but for different customers. Pseudo-wires will act like "services within a service", since you don't need to configure anything more.
This setup has a lot of potential uses - Think about HA-traffic for example - Firewall clusters, routers, databases etc. Every two things that just need a cable between them, but unfortunately aren't in the same datacenter - All these can be solved with pseudo-wires. VmWare host networking comes to mind also. And of course older protocols that isn't really IP, but happen to use a networking cable - Perfect case for pseudo-wires. There's probably a lot more cases that I can't think of now. 

Connectivity Fault Management, or in short - CFM, and also known as 802.1ag, is a standard that sort of adheres to a pseudo-wire. It simply gives you the opportunity to make sure that the pseudo-wire service behave as a real cable: It will replicate the port states so that one port can't go up unless the other port is connected, just like a real cable. This can come in handy for services like HA-setups in firewalls for example, since they might decide the failover based on the link state of the HA interfaces. Having CFM on these ports can drastically improve the failover times, since it will instantly bring down the pseudo-wire port in the other end if there is a link failure. If you don't configure CFM, the HA-setup has to probe other end first before deciding that there is an outage.\\
__It is considered best practice to always use CFM when deploying a pseudo-wire.__

SPB is all about bridging - Everything is bridged all the time. So how do you route? Let's find out.

[Shortest Path Bridging Part 5 - Routing](https://networkundertaker.com/2023/04/12/Shortest-Path-Bridging-part-5.html)

[Start page]({{ '/' | absolute_url }})
