**Shortest Path Bridging Part 1 - Wait, what? Why?**

Well, that IS the question isn't it? SPB connoisseurs might oppose - They would argue that the real question here is why you haven't seen nor heard about SPB being used ANYWHERE by ANYONE in your whole professional career. If you have, you are as rare as a combat unicorn really. I'm more of a white rhino myself.

This gem of a technique - also known as 802.1aq - became something of the "Betamax of networking" - It is great, and it works just fine, but it isn't really supported by anyone. The last bit is something of a stretch - It is present in the Linux kernel, it is supported in OpenVswitch. But on the hardware side - Only Extreme (via the bits bought out from Avaya), Nokia SR, and Alcatel-Lucent Enterprise have working SPBm (m for mac, more about that later) implementations as of 2023. Maybe Huawei too. Not counting them. But anyway - SPB isn't really supported by any of the big vendors in the business. EVPN/VXLAN won the race by a thousand to one (if you count the number of deployments of the techniques, respectively). To say that Shortest Path Bridging is anything but a very niche technique would be a lie. But it's out there. It isn't going away anytime soon. There are reasons for it. 

So what is SPB exactly? In its most basic form, it is a safe way to stretch L2 over ... Something. We'll get back to that. I've heard more colourful descriptions like: "Q-in-Q on speed". That isn't technically correct, but it does have a nice ring to it. I would actually add to that: "Q-in-Q + mac-in-mac encapsulation on speed" (speed in this case of course means "no blocked paths", and nothing else). Digging further down into what actually takes place in your equipment, I would say "tunneling". Get used to that way of thinking. There's more to SPB then "just" L2 stretched over... whatever it was. It also depends on your views on that topic really. Tunnels have mesmerized people since the days of Brunel and Stephenson. But let's not start a fist-fight over what a tunnel really is, (Hey, is MPLS tunneling by the way? \*Dodges fist\*), at least not yet. Let's check out what you can actually do with SPB.

I have played around with trying to explain this as short as possible, to really boil everything down to one sentence. It is a good way to stop babbling (in which I have to practice). 

For good or worse, here it is: The ONE sentence to explain it all: 

>> __Shortest Path Bridging is a link-layer underlay/overlay transport technique based on IS-IS TLV extensions that can scale up to thousands of nodes, and that gives you the opportunity to create L2VPNs, pseudo-wires, L3VPNs, and to handle inline routing and the propagation of routes within your routing domains over a redundant backbone network where all links are active.__

If you read that out, now is the time to stop and breathe for a few seconds. No fainting is allowed. That IS a mouthful. But it does sound an awful lot like MPLS/VPLS, doesn't it? That is because Shortest Path Bridging solves many (not all though) of the same problems MPLS does, just in another way. It also overlap the functions EVPN/VXLAN offers. And that is because SPB really is the other branch of the EVPN tree - It offers more or less the same functionality, but with maybe 5% of the configuration EVPN/VXLAN needs. 

SPB is very very simple to configure - Mostly because it doesn't need a complex stack of collaborating protocols like MPLS and EVPN/VXLAN does in order to work. It only needs one: IS-IS. This is what SPB relies on for both its underlay and overlay: It is used for building the loop-free multipaths in the underlay, for source-learning, and for carrying VPN-routes across the backbone. All done with a few clever extensions added to the IS-IS TLVs: RFC6329 - IS-IS Extensions Supporting IEEE 802.1aq Shortest Path Bridging (D. Fedyk et al).

The RFC describes two operational versions of SPB - SPBv (v for vlans) and SPBm (m for mac-address). SPBm won - All working implementations as of 2023 are SPBm, which uses PBB (Provider Backbone Bridging, 802.1ah). PBB is a mac-in-mac encapsulation technique, which in SPBm is the service delivery data plane - The overlay where you define your services. And you only need to define the service on the nodes that have them defined on a client-facing port. All other nodes in the path will switch the traffic transparently. The user traffic in the SPB environment will be forwarded to its destination based on the information in the PBB mac-header, within the BVLAN and via the corresponding SPF tree topology created by IS-IS. 

When the customer traffic arrives at the ingress SPB edge node (BEB - Backbone Edge Bridge), a PBB-header that contains the B-VID (BVLAN-id), the unique IS-IS service-id (ISID), and the B-SA and B-DA (backbone src-mac and backbone dst-mac) is wrapped around the frame. The traffic is switched within the SPB environment based on its PBB-header only. 

When the traffic exits the egressing SPB-node, the PBB header is stripped off, and the service payload is delivered to the SAP port. An SPB node not terminating a SPB service is called a BCB (Backbone Core Bridge), and it only switch frames to their destination based on the destination mac in the outer PBB mac-header within the BVLAN. So, a BCB is totally ignorant of what kind of payload it actually switches - There is no need for it to learn any mac-addresses deeper within a service if it doesn't terminate that service. No lookups, no source-learning. Just switching. 

It is pretty close to how label switching works, to some extent. Just as MPLS, SPB makes it possible to forward all kinds of traffic in a simpler, and more resourceful way, at scale.

Shortest Path Bridging was created with co-existence with the legacy network environment in mind. You can deploy SPB in any existing switched environment, alongside of legacy VLANs, providing a totally separated network within SPB. 

The following prerequisites have to be in in place:

* Jumbo frames. But only slightly bigger then you already had for your VLAN tags. 1524 is needed due to the mac-in-mac PBB encapsulation. Might be a PITA to enable in some environments. Not in ALE (Alcatel-Lucent Enterprise), since everything is 9216 default. More vendors does the same thing now, like Arista. (Why not have jumbo frames enabled default on all ports in 2023? It doesn't hurt anyone.)

* Non-SPB switches must be able to switch the mac-in-mac frames transparently. All major vendor's switches after 2010 can do that.

* It is best practice to disable STP on the BVLANS (Backbone VLANS) that pass through non-SPB switches. The BVLANs in the SPB-switches will drop all BPDUs anyway, and STP is default disabled on all BVLANs in SPB-enabled switches. The SPB control plane has to be in control of all paths included in the setup. A BVLAN also have its source-learning disabled. Yep. All src-learning is done at the edges, in the customer-facing services, and is handled by the IS-IS control plane. So you just don't want anything else in your BVLAN to decide if a link should be open or not.

IS-IS doesn't ride on top of IP like OSPF or BGP by the way. It has its own IP protocol: 124. So SPB is using the IS-IS control plane messaging, very much like how MP-BGP extensions is handling the EVPN underlay within the BGP protocol. 

__In practice, an SPB node doesn't need any IP set for SPB to work, even when providing L3 services to IP packets.__ 
(Although some losers seems to like to have a management IP for some reason...).

In part 2, we will set up the BVLANS, enable the SPB service, and add links, effectively creating the SPB backbone/underlay.

[Shortest Path Bridging Part 2 - Setting up the backbone](https://networkundertaker.com/2023/04/10/Shortest-Path-Bridging-part-2.html)

[Start page]({{ '/' | absolute_url }})
