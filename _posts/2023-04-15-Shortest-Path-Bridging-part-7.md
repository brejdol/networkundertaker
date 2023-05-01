**Shared Services VPN and route leaking**

In a highly segmented L3 VPN design where every VPN maps to its own VRF, it is still common to have shared services that many VPNs have to reach. For example DNS, internet access, DHCP, NTP services etc. One way of doing this is by using VRF leaking.

![Shared-Services](/Shared-services-L3VPN.png)

In the above example, we have two customers, A and B. Both customers have a VRF in all four BEB nodes, customer A has isid 4002 in the SPB backbone, customer B has isid 4003. Routes are propagated between the customer sites/VRFs with the L3VPN setup we explained [here.](https://networkundertaker.com/2023/04/12/Shortest-Path-Bridging-part-6.html)

BEB1 and BEB2 also have VRFs containing the shared services firewalls that handle the internet peering with eBGP towards the upstream provider. The shared services VRFs is connected to the backbone via isid 4004. Customer A and B needs internet access. To achieve this, we can leak routes between the customer VRFs and the shared services VRF, since everyone is connected to the SPB backbone:

```
vrf shared-services ip export route-map ebgp_routes_only
vrf Customer_A ip import vrf shared_services all-routes
vrf Customer_B ip import vrf shared_services all-routes
vrf shared_services ip import vrf L3_VPN_A all-routes
vrf shared_services ip import vrf L3_VPN-B all-routes
vrf shared_services ip import isid 1002 all routes
vrf shared_services ip import isid 1003 all-routes
spb ipvpn redist source-vrf shared_services destination-isid 1002 all routes
spb ipvpn redist source-vrf shared_services destination-isid 1003 all routes
```

Let's break it down what goes on here, since AOS is completely incomprehensible:

First, we export only the eBGP routes from the shared_service VRF (with the help of a route-map, (creating one is easy, but outside the scope here).
Then we import the exported routes into customer A and B.
The same in the other direction - We import the routes from the local customer A and B VRFs into the shared_services VRF.
We then import the routes from their other sites by importing all routes from the customer's ISIDs.

Finally, from the global route manager (not within a VRF), we redistribute the exported eBGP routes from the shared_services VRF into the customer A and B's ISIDs so that all connected sites get them.

Well, this isn't completely incomprehensible actually. And this is where SPB shines - You can do a lot with little. Config is short, readable, and possible to grasp fairly quick. It isn't nestled, nor obfuscated. You can see what the outcome will be. 

If you use automation, you don't really care about how long or how nestled the configuration is, and rightly so - You want to concentrate on more important things. But in some cases, for example when you have to troubleshoot, the actual bulk of the configuration does makes a difference. 

And day-to-day operations and troubleshooting is up next: 

[Shortest Path Bridging Part 8 - Operating the backbone in SPB](https://networkundertaker.com/2023/04/17/Shortest-Path-Bridging-part-8.html)

[Start page]({{ '/' | absolute_url }}) 