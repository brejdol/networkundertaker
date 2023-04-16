**Routing continued - L3-services**

![L3-service](/L3-service.png)

In the above example, we have four different BEBs belonging to the same customer, each with a VRF containing a local user network where the default gateway is in the VRF. Each VRF also have a "WAN" interface each in the same subnet within a single SPB service that connects the four sites. Routing is only performed at ingress/egress VRFs, the traffic between the VRFs are bridged. On the WAN-side - Everything is a single L3 hop away regardless of how many L2 hops (BCB) there are in between.

The routing between the different VRFs can be done in two ways:

**VPN Lite**

A VPN Lite L3 service is created by overlaying a routing protocol on top of the SPB service that connects the VRFs. The routing protocol can be static, BGP, or OSPF. The routing protocol will be a specific instance for each VRF. This will work fine, since the SPB service that connects the VRFs is a directly connected network - OSPF for example, will work normally and find neighbors, create adjacencies etc. The backside of this is that you have to run a routing protocol, with the extra config, complexity, and CPU load this will carry with it. You can do it all in SPB instead.

**L3 VPN**

A SPB L3 VPN will instead use the existing IS-IS instance to carry customer VPN routes without the need of an additional routing protocol. This extra "metadata" is added into the IS-IS TLVs extensions. Each customer/tenant will still be bound to its own VRF, and the IS-IS TLVs map to the customer's service ISID, providing L3 isolation between different customers. This means that SPB matches the same functionality as in MPLS and EVPN, where OSPF or IS-IS is used for backbone node reachability, and  MP-BGP for customer VPN route transport. In an SPB L3 VPN, IS-IS can do both. Having just one protocol instead of two (or more, for example LDP in MPLS, VXLAN for bridging in EVPN), lowers the complexity and the sheer size of the config (which is huge in EVPN/VXLAN). The results is a network that is very easy to deploy and to maintain. The configuration is simple, self-explanatory, and easy to understand.

SPB bears more resemblance to MPLS then to EVPN/VXLAN though, since a SPB BEB node more or less plays the same role as a MPLS PE node, and a SPB BCB node is similar to a MPLS P node. All SPB BCB nodes will only switch the backbone BVLAN. The don't have to learn any customer VPN routes nor have any VRFs.

A SPB L3 VPN doesn't need any additional routing protocol. VRF routes are instead exported to the global SPB IS-IS instance, mapped to the customer's ISID, and bound to the WAN IP as a gateway. The other BEBs will import those routes into their local VRF routing table, and have their WAN IP as a gateway address. IPv4 and IPv6 works in the same way. Route-maps can (and should) be used.

```
service 3000 spb isid 3000 bvlan 4001 
service access port 1/1/51A 
service 3000 sap port 1/1/51A:0
service 3001 spb isid 3001 bvlan 4001
vrf create Customer_A
Customer_A:BEB-> ip interface LAN address 172.16.92.1/24 service 3000
Customer_A:BEB-> ip interface L3VPN address 100.64.90.1/24 service 3001
Customer_A:BEB-> ip export all-routes
Customer_A:BEB-> ip import isid 3001 all-routes
exit
spb ipvpn bind vrf Customer_A isid 3001 gateway 100.64.90.1 all-routes
```

The above config first create a new service 3000 used for the LAN-network, and this is presented untagged on SAP port 1/1/51A. The second service 3001 is used as the internal SPB L3 VPN interface. The customer VRF is created. IP is set on the interfaces, all local routes in the VRF is exported, and all incoming routes via the L3 VPN on service 3001 is imported. The exit command takes you out of the VRF, and finally, we map the L3VPN service to the vrf and set the gateway to the IP of the L3VPN interface, and set that all-routes should be handled (a route-map within the VRF is where you should filter imported/exported routes). This is fairly straightforward and possible to grasp. The only thing I struggled with is that you set the gateway to your own L3VPN interface IP. But it is logical if you think about it. That IP __IS__ the gw to all the other VRFs behind it.

A SPB L3 VPN does use the onboard resources better, since everything rides on top of the already running SPB IS-IS instance. Even if the amount of customers grow over time, it is still the same instance. If we compare this to instead running VPN Lite with a routing protocol in all VRFs instead, scale quickly becomes an issue. 4 customers spanning services in 8 BEBs require 4 OSPF instances per node. If dual stack is needed, we're at 64 OSPF configurations. Dual stack in SPB is still only one SPB IS-IS instance, since IS-IS doesn't depend on IP itself, and is IP agnostic forwarding wise anyway - __IPv4 or IPv6 doesn't matter.__ 

If your environment grows, this can make all the difference. IS-IS signalling is also very quick - convergence will be faster with only one protocol running. With all this said - L3 VPN is only possible within the SPB domain. All dynamic routing that is external must be handled with the appropriate protocol in the BEB VRF. But within the SPB domain, I can't really find many reasons to NOT use what's already there.

In the next chapter, we will stay on the L3 track -  Shared services and route leaking.

[Shortest Path Bridging Part 7 - Routing Continued - Shared services and route leaking](https://networkundertaker.com/2023/04/15/Shortest-Path-Bridging-part-7.html)

[Start page]({{ '/' | absolute_url }})
