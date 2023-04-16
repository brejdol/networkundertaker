**Routing in SPB**

SPB/IS-IS doesn't use IP for its core functions. Many deployments are setup as a transport service that delivers its services to customers transparently. An IP-free core is more or less the norm. So how do you route? You just do it:

```
ip interface service-1000 address 172.27.19.1/24 service 1000
```

If it would be a legacy VLAN, you would do it like this:

```
ip interface vlan-1000 address 172.29.19.1/24 vlan 1000
```

Yes, this weird AOS syntax again. It's not that hard to understand though. The name on an IP-interface can be anything, but a good practice is to name the VLAN/Service to something self-explanatory. Anyway, you can do inline routing in SPB in the same way you would do it in a legacy VLAN. The same goes for VRFs and all routing protocols and commands:

```
vrf create usr
vrf usr ipv6 interface "service-1000" service 1000 ra-send no
vrf usr ipv6 address 2001:db8:72:19::1/64 service-1000
vrf usr ip interface service-1000 address 172.27.19.1/24 service 1000
vrf usr ipv6 bfd interface service-1000
vrf usr ipv6 bfd interface service-1000 admin-state enable
vrf usr ip router router-id 0.0.0.90
vrf usr ipv6 static-route ::/0 gateway 2001:db8:72:22::fe
vrf usr ip static-route 0.0.0.0/0 gateway 172.72.22:254
```

Bam! I created a VRF named "usr", added an IPv6/IPv4 interface on service-1000, turned off router advertisements, enabled BFD, set a router-id, added a static route for IPv6/IPv4. OSPF, BGP, IS-IS, RIP, static routes, you name it. It works exactly the same as with a legacy VLAN. The traffic only need to pass the switch one time, this is why this is called **Single Pass Inline Routing**. 

You can also mix legacy VLANs and SPB services in the same vrf, and route between them seamlessly. However, there might be reasons to keep the routing part outside of SPB. You might have legacy routers that will perform the routing in which case the SPB network only acts as a transparent transport. You might want to keep the transport network and the routing separate. Another reason might be because your are using specific functions that doesn't work if you route within SPB, in fact, I have such a case right now: 

A client use port mapping extensively (port isolation/private vlan) in their environment. They have large L2 domains kept in place by port mapping in their core out to their nodes, blocking lateral traffic on L2 between the nodes. It is a legacy setup, and maybe not up to par with how you would design things 2023, but nevertheless - The customer don't want to change this design, but still use SPB with all its features and redundancy. The problem is that port-mapping is impossible to do between SAP ports. Then you are either forced to start using ACLs very extensively in a way that doesn't really scale, or you have to move the routing part OUT of SPB. With a bit of trickery and a loopback cable, you can route VLANs the legacy way and use SPB at the same time.

**Two-pass routing with external loopback**

In this setup, we will separate the routing (which will be done as legacy "dummy" VLANs) and the SPB-domain. Check this out:

```
vrf create srv
vlan 110 name vlan-110-dummy
vlan 111 name vlan-111-dummy
vrf srv ip interface vlan-110-rtr address 172.19.35.1/24 vlan 110 rtr-port port 1/1/1 tagged
vrf srv ip interface vlan-111-rtr address 172.19.36.1/24 vlan 111 rtr-port port 1/1/1 tagged
service access port 1/1/2
service spb 1 sap port 1/1/2:110
service spb 2 sap port 1/1/2:111
```

This creates a two-pass routing setup (traffic pass the router 2 times), where we have the dummy VLANs 110 and 111 with IP interfaces in the VRF named "srv", and the interfaces maps to the SPB service 1 and 2. The rtr-port option turns off all L2 protocols, like STP etc. A loopback cable between port 1/1/1 and 1/1/2 bridges the services with the dummy VLANs. It is also possible (read: Best practice) to use a linkagg in this setup for redundancy.

Next up, we'll take a look on L3 services.

[Shortest Path Bridging Part 6 - Routing Continued - L3 services](https://networkundertaker.com/2023/04/12/Shortest-Path-Bridging-part-6.html)

[Start page]({{ '/' | absolute_url }})