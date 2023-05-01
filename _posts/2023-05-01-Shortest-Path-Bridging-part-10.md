**Messing up SPB**

What? Well, my highly personal view is that you need to know all limits regarding just about everything: Your need to know your own limits, and you need to know the tools you use inside out. I think it's only fair to be very specific with what you can do, and what you can't do with SPB. Otherwise this would just be some sort of SPB cult/fan-page. And it really isn't. SPB is just a tool, just like a hammer. But every problem isn't a nail. 

I have around three years of operational knowledge with SPB wich means I've made numerous mistakes, solved a whole lot of problems, I've done extensive troubleshooting, and I have broken stuff. Badly... So, a few do:s and don't:s might come in handy:

**Stuff you can get away with:**

* You can pull out an uplink on a switch at any time, as long as you have more then one, and as long that you don't have issues with your SPB mesh (obviously). No one will notice. For real. Failover is usually 50ms or below. I haven't measured, but it is shorter then any regular user can notice. A firewall HA failover - wich most of my customers perform during work hours since it's very quick, and very seldom noticable, even if you are in a call - is way slower. You can have a brief moment during a HA-failover (still sub-second) of silence in your call etc. Not so with SPB. It is something like 10x faster. The same goes for the other way around - Adding links doesn't trigger any topology change you will notice, it just adds another path. Just remember to set your link costs properly if mix different link capacities. You can read more about that here. --> [Shortest Path Bridging Part 2 - Setting up the backbone](https://networkundertaker.com/2023/04/10/Shortest-Path-Bridging-part-2.html)

* You can, if you tread lightly, setup your entire new and shiny SPB network within your old network, then slowly migrate away the old network as you go, one link at a time. I've done this. You can also create a terrible mess. I've done that one too. More about that further down.


**Stuff you probably won't get away with:**

* Looping a service is very possible. SPB is by itself very compartmentalized - It is built with the intent that what happens in customer-land (ie: service-land) stays in customer/service-land. That is mostly true, but not entirely. SPB by itself doesn't have many protective measures. Your weak spots will always be the customer side. It is considered best practice to always enable loopback protection on all SAP ports. It works like STP, but only within one switch. That will protect the environment from simple mistakes, like putting a cable between two SAP ports that have the same tagged services, or an untagged service. If you do that with a couple of 100G interfaces, it will hurt. I've felt the pain... Your SPB environment will not go down by one loop on the service side though, regardless of the size of it. In fact, you will not really notice unless you have IP interfaces configured on the services. Then you WILL see a CPU impact. Otherwise, it will be switched transparently in the hardware. You can see that the interface utilization goes up really high. That customer will be out though. 100G can and will hurt.

* Create a mess with old the legacy network using STP, and new links over SPB. You really need to plan this. Adding the BVLANs in your old environment provides paths to SPB that you might need initially to create the mesh. To STP, the new paths through SPB just looks like you suddenly got a lot more fibers. You MUST turn off STP for all BVLANS in your legacy environment, otherwise you will have blocks where there should be none. For SPB, the path through the legacy network is invisible, it can only see SPB nodes. 

* Try to segregate so that you don't have the same VLANs as services on the uplinks in both new and old environment. This can be tricky to pull off, because you have your set amount of switchports. Just be very aware of that STP can create new blocks where you didn't have any yesterday. Try to make a plan where you connect new uplinks to the legacy environment via SPB. Then you can move one VLAN at a time from your old uplinks to your new SPB-based uplinks. 

* You will create a mess if you mix SPB uplinks with legacy uplinks. STP will work, but you can end up in weird situations where you have to manually turn off STP on a legacy VLAN in order to make it work. We referred to this as being in "the no man's land" - Basically, your network state is... unclear at best. You will get unexpected blocks, and it will be hard to instantly see why, and if you disable STP, you might/might not create a loop. Unclear status is the worst. Do everything to avoid it.

* Looping your BVLAN. Just don't. Make sure to turn off STP in your legacy environment if you switch any BVLANs through it, and don't ever connect anything else to the assigned BVLANs. 

**Stuff SPB can't do**

* Active-active dual homing. I don't understand how that would be possible. It maybe is, but I think not. Will get back to you if I find a way to pull it off. As of now, if you need A-A dual homing, you'll have to go the EVPN/MPLS/SR route.

* Interaction with the outside - SPB is based on an IGP. You can't easily interact with anything else, unless it speaks SPB. You can interconnect with an Extreme environment for example. But otherwise, not much talk SPB natively. You will have an edge to the outside in most cases. ProxMox is the only virtualization environment that I know of that can handle SPB. Haven't tried it though.

* QoS. Well you can do a little. But SPB is for the datacenter, all pipes are meant to be open at full blast.

* Traffic engineering. You can do that to some extent if you fiddle with the priority. Usually, you don't do that. The whole point is to let the protocol solve that for you. You can have path assymetry within SPB, but all that is obfuscated from your view. SPB isn't MPLS. You don't have all those knobs to turn. SPB is simplistic. Maybe too simplistic for your needs.


[Shortest Path Bridging Part 11 - Summary](https://networkundertaker.com/2023/05/01/Shortest-Path-Bridging-part-11.html)

[Start page]({{ '/' | absolute_url }})