**Introduction**

<p>I have run an IPv6-only PoC-network for 15 months, testing out how different clients and operating systems works, and thought I should share my findings. There are a lot of articles about IPv6 out there, but few that walks you through the entire process from native IPv4, via dual stack all the way to the goal:</p>

* A native IPv6 network that transparently translates traffic to legacy IPv4 destinations on the fly when needed. 
* A native IPv6 network that automatically can handle legacy applications that need native IPv4. 
* A native IPv6 network that just works, regardless of what you do or work with. Ideally, you wouldn't even know you're using it. 

Legacy applications that need a literal IPv4 address on the client are still plentiful in the enterprise and in OT environments, and they will not go away anytime soon. If you have such dependencies that need to be handled _manually_ on every unit, this will be hard if no automation exists, it's as simple as that. If you have 10 units, it will work, but not if you have 10.000. It _might not_ be possible/feasable to deploy IPv6 _at all_ in certain networks. Embedded devices might be hard-coded for example. This is how it goes. If it will not work at scale, why bother? Draw your own conclusions. Keep it real. And read on.

This write-up assumes that the reader has a basic understanding of IPv6 as well as DNS/DHCP, DNAT/SNAT and routing in general. A collection of links for those who want to know more is provided at the bottom of this page.

(TLDR; Native IPv6 works fine for a vast majority of the existing clients and servers out there, with a minimum of manual actions client-side, but sadly the Windows environment still (as of dec. 2023) lacks an important piece of the puzzle that _might_ be a deal-breaker.)

**Background**

It might not be at the top of everyone's mind, but let’s start with where we want to be with our IP addressing. IPv6 has been around for around twenty years. It is the successor of IPv4, for better or worse, like it or not. It really is what we should deploy and use everywhere. But we don’t. We rewrite packet headers instead, sometimes many times in a row between a source and the destination in order to handle IP clashes, or addresses that simply are unroutable on the internet (i.e: addresses from the CGNAT-scope, RFC1918 etc). However, this _usually_ works fine. NAT has saved more butts than pants. I for sure know that my butt would have been on the line many times without it. NAT is a splendid tool, the ”duct tape” of networking that holds everything together. 

Or is it? The problem is that we sorta maneuvered into a global situation where NAT is the _only_ readily available solution to an endless amount of addressing issues - In the enterprise, in the wan, in the cloud. There isn’t enough IPv4 addresses to provide end-to-end connectivity everywhere to everyone. The base intent of the internet failed just a few months in when it was made public. It was _obvious_ that this wouldn't scale for long, something had to be done. 

In 1994, the first year modems were readily available for the home users here in Sweden, we saw the humble beginnings of NAT - Which by the way _isn’t_ a swiss army knife of networking, it exists solely because of the global issue with usable address-space. Addresses had to be saved. No one had enough addresses, not even the entire world. NAT is a tool, just like a hammer. But a lack of enough addresses isn't "nails".

_Is this really where we want to be with our addressing?_

A more apt question might be:

_Is this where we want to stay with our addressing forever?_

It sure seems that way, at least sometimes. But what if it's possible to do it the opposite way - Where IPv6 is the new default, end to end connectivity is the norm. Where IPv4 is the oddity, the exception, the old address space you’ll translate _to_ when needed? 

Where IPv4 really _is_ legacy IP, still there just as a reminder of the errors made by an earlier generation that couldn't count high enough. 
It is _very_ possible to get there, today even. Bear with me.

**Getting to it - Acceptancy**

Ok, let’s say you accepted your fate as a person in networking - You understand that IPv6 eventually will be of importance, so you learn how it works. You learn to count and make sense of hextets instead of octets. You see that everything (almost) works very much like v4 regarding routing. (”Nice, I totally get this!”) All the crazy stuff seems to happen on the local link (”Hey why on earth do I have like three different addresses on the same interface?!”). 

Fast-forward a couple of years. You now know your stuff. You can walk the walk, and talk the talk.  You can subnet into /63 and /71 without missing a beat, and you understand why you never should. Hurricane Electric thinks you are an IPv6 Sage, but you don't believe them. You test stuff. You do mistakes, and you fix them. You make certain observations regarding IPv6:

* You finally have all the addresses you will need for the forseeable future.
* End-to-end connectivity means your pipes are ... straighter? It sure feels that way at least.
* Dual stack can really be twice the work or more to maintain, rule/routing-wise.  
* People don’t like a broken IPv6-network, or you for that matter
* It would be grand to just use IPv6 and drop that other crap, whatever they called it

**On the way to native IPv6**

So, we have this little (or big) network. It has routers and firewalls, it is segmented on both L2 and L3. It contains clients, servers, users, guests, you name it. For this part, we will focus on just one network, and follow its path from IPv4 only to native IPv6 with translation to IPv4 when needed. Let me introduce you to Joe:

![Joe-IPv4](/joe-ipv4-1.png)

Joe is a very pale guy that turns kinda blueish in the wintertime. He just got a new cellphone, the mighty iPhone 902 Super MAX. It weighs 200 pounds. He connects to the office wireless network for phones, using an authentication method uknown to us. He checks his flight to Vegas, reads the DM his daughter sent him on Instagram, and proceeds with the rest of his day. 
The network he uses resides in the company's internal firewall, which provides him with an IP/DNS via DHCPv4. The internal firewall has a default route to the perimeter firewall. The perimeter firewall has a default route to the ISP's router. The perimeter firewall does SNAT which translates Joe's internal 172.18.142.x IP to a public IP, and off he goes onto the internet. So far so good.

We now rewind the tape and start over. It's the same blue Joe, with the same very heavy phone. But this time, the networking slaves in his company managed to type in the correct IPv6 IP on the interface, and setup the firewall rules, routing and DNS for IPv6:

![Joe-Dual-Stack](/joe-dual-stack1.png)

This time, Joe gets the same IPv4 address and DNS via DHCPv4, but he also gets an IPv6 GUA address (documentation prefix in this example) and a RDNSS (company DNS server) via SLAAC/RA. He checks his flight to Vegas, and since the airline company web page doesn't have an AAAA DNS record, only an A record, this traffic follow the above IPv4 path exactly. He then checks the DM his daughter sent him on Instagram, and since Instagram is reachable over IPv6 with a AAAA record, his very heavy phone opens an IPv6 connection to the IPv6 IP it got from the local resolver. The traffic follows the default route from the internal firewall, via the default route in the perimeter firewall and onwards. No NAT is performed because it isn't needed. End-to-end connectivety. And all OS:es prefer IPv6 _if_ they have an IPv6 address. 

The company networking slaves gets an idea: This network for company cellphones might be a good candidate for native IPv6, due to the fact only fairly new and company managed cellphones are using it. They don't have any dependencies to legacy IPv4 applications, but they obviously will need some sort of conversion mechanism when trying to reach IPv4-only company/internet resources. 

**NAT64/DNS64**

NAT64 does what is says - It translates IPv6 addresses to IPv4 addresses. DNS64 is another beast entirely. It is a DNS proxy that looks at all DNS queries from the clients, and when an IPv6-only client tries to reach a fqdn that doesn't have a valid AAAA record, the DNS64 proxy will create an AAAA DNS record from the A record, and it will map this record to an IPv6 IP. 
There is a reserved IPv6 prefix that can be used for this translation: 64:ff9b::/96. A /96 prefix leaves 32-bits for addresses, and that is what we need - The entire IPv4 address space from 0.0.0.0-->255.255.255.255 fits there. 
So, the end of this IPv6 address will look like this: 64:ff9b::(ipv4-address-in-hex). Traffic to this mock AAAA record/IP will then be translated into an IPv4 address with NAT64 and then follow the IPv4 path to the destination. This whole process adds a few milliseconds on all lookups, but it isn't really noticable, it doesn't feel sluggish at all, at least not in my setup.

**How it is done in a Fortigate firewall**

You can do this in _anything_ that can handle DNS64/NAT64, and the setup will be sort of the same, just with different methodology depending on the gear. I just happen to work with, and have access to Fortigates:

* In the internal firewall where the gateway of your IPv6 network lives, turn on the DNS Database feature, enable DNS service on the interface, use "Forward to system DNS"

![Enable-DNS](/enable-dns.png)

* Make sure the rdns in the SLAAC settings,  or DNS option in DHCPv6 is pointing to the interface IP. The DNS proxy will reside here. 

Create an IPv6 VIP, tick "Use Embedded", and set the external IP-range to use the last 32 bits of the address: 64:ff9b::-64:ff9b:ffff:ffff. This ensures _any_ IPv4 address in the entire address space can be mapped in this dynamic VIP. The destination side of this DNAT VIP will get the "real" IPv4 address dynamically from the DNS64 process. You don't need to touch anything else on the VIP, it is done.

![DNAT64-VIP](/DNAT64-VIP1.png)

You already have rules for your IPv4 and your IPv6 traffic since you had dual stack running. Now, add another rule, where the src-ip is your IPv6 /64 prefix for the network, and the destination is the VIP you created. Also, create a SNAT-pool with a few IPv4 IPs that will work with your routing table. Make sure you enable NAT64 in the NAT section:

![DNAT64-Firewall-Policy](/dns64-fw-policy-snat-pool.png)

This rule will make this special VIP reachable, and the traffic that hits this rule will get NAT64:ed out on one IP in the SNAT pool. For the perimenter firewall, the SNAT pool’s IPs looks like source IPs, it's just ...SNAT. 

```
!!!
Make sure the IPs in the SNAT pool starts and ends on valid subnet boundaries! 
Otherwise you will get unpredictible results!
!!!
```

If everything checks out, you can now enable the DNS64 proxy in the Fortigate CLI.  
This turns the whole setup _ON_:

![Enable-DNS64](/enable-dns64.png)

If you enable "always-synthesize-aaaa-record", the DNS64 proxy will do this for _everything_, regardless of an AAAA record exists or not. This is unnecessary in our setup.

You now no longer use IPv4 for anything that has a DNS entry, since the Fortigate will generate AAAA-records out of the A-records and then map that AAAA to a 64:ff9b::xxxx address, and then do NAT64 on that traffic on the way out. 

But you are still dual stacked, you still have an IPv4 address, don't you? That _can_ be awfully handy sometimes, especially if you want to open an IPv4 socket to something that has an IPv4 address, but no DNS record. If you're a networking person, you'd might like to be able to _ping_ an IPv4 address for example. But now, let's up the ante. We’re not here to stay in the safe zone. 

**DHCPv4 option 108**

The DHCPv4 option set is just additional metadata that will be sent to the client together with the IP address. You can inform about DNS, gateway, NTP-server, vendor specific options etc etc. Option 108 is not widely known though, and it is rarely used. What does it do? Well, it tells the client that just received an IPv4 address, to _turn off IPv4_ for (value in hex) seconds. Yes, that's right. This option exists for this specific setup where you really want to make sure that the client _doesn't_ have an IPv4 address _at all_. But we're good to go. Let's set it:

![DHCP-Option-108](/dhcp-option-108.png)

As you see, I did turn off IPv4 for 4,294,967,296 seconds. Because I could. Any value slightly over your dhcpv4 lease-time would be fine. 

```
!!!
Make a memory note of the importance of always blocking dhcp acks/servers on client-facing ports.
Option 108 is great way to DoS an IPv4 network without anyone understanding what happens. 
Clients will not renew their IPs unless network is restarted on them. 
You would have to use Wireshark in order to actually _see_ the option at play.
!!!
```

If you set this option and reconnect to the network, and then check your IP on your iPhone, you will see something weird after a few seconds (pardon the swedish):

![iPhone-IPv4-CLAT](/iphone-ipv4CLAT2.png)

You see that you've grabbed 2 IPv6 IPs - That's good and well. But your IPv4 address lines will be empty for a few seconds, then they will be populated with a weird IP/Mask and gateway that you haven't set. You haven't set anything, you just turned IPv4 off, didn't you? Yeah, you did. Usually, a client that doesn’t get a DHCP lease within 20-30 seconds will hand itself an APIPA address (169.254.x.x) in order to at least have some address on the local link. But in our case, we actually _did_ get an IPv4 address from the DHCP server, albeight with the option 108 that ordered us to shut down IPv4 at the same time. This is something else. 

**CLAT/464XLAT**

What you see now is actually the CLAT client doing its magic. This is the _customer-side translator_, locally on your phone. It is doing stateless NAT46. The CLAT in the client together with the PLAT (provider-side translator) which in our case is the NAT64 together provides the translation mechanism know as 464XLAT.  
The CLAT client is triggered by the absence of a IPv4 address, but IPv6 exists, and there is a NAT64 prefix available (signalled via router advertisement containing the PREF64 option, or via the ipv4only.arpa fqdn in DNS). What the CLAT client does when triggered is:  
* Set an IP/gw out of the reserved range 192.0.0.0/29
* Create a "VIP" on the gateway, doing stateless NAT46 --> Interface IPv6 IP.
* In the other end, the NAT64 setup will reverse this traffic back to IPv4 and send it out SNAT:ed, hence the 464XLAT name of the setup in total.

What does this mean then? Well, it means a fairly updated device (from OSX 13 and iOS 16 and up, Android 4.3+, including Chromebooks, etc) has a means of handling the fact that it _doesn't_ have an IPv4 IP.  
This functionality is a "helper" for applications that need a literal IPv4 IP on the phone itself - For example when pinging an IPv4 IP from the phone, or using a legacy app that _only_ can use an IPv4 address (which isn't really gonna happen due to the fact that Apple forced all apps in appstore to support native IPv6 since 2010-ish).  
Think of it as an automatic NAT46 function within your phone that needs no configuration at all. A working CLAT client onboard a device means that it will work very well on an IPv6-only network. It will just work as usual.  
As a regular user, you simply will not notice that you don't really have a "real" IPv4 IP _at all_. The same goes for _all_ client devices that have a CLAT client onboard, heck, even servers will work just fine. You _can_ open an IPv4 connection to an IPv4 address, even though you _don't_ have a "real" IPv4 address.  
A regular linux client/server doesn't come with a built-in CLAT client, you'll have to install Tundra or something similar, and run them in CLAT mode (not covered here).  
It's simple enough but it might not scale if you have embedded devices that are hard to handle. But now we're getting to the elephant in the room, and it is a big elephant:  

The Windows suite of operating systems doesn't have a working implementation of CLAT that can be used on wired/wireless interfaces as of today (dec. 2023). I don't know of any tools that can fix this either. To be honest I haven't checked for 3rd party tools for this, since this _should_ be handled in the OS in my opinion. The irony in this is that Windows actually _has_ a CLAT client, but it can as far as I understand it only be used on WWAN type interfaces.  
This is pretty unfortunate given the amount of Windows clients/servers that are out there. As of now, there is nothing else to do but wait for this to be enabled by default on windows. Or at least give us an ON/OFF button for it? When this finds its way into windows, we're talking. But back to our pale blue friend Joe. How is he doing?  

He's doing alright, but his arm is swollen, and his back hurts. He hasn't noticed any difference in how his phone works, even though he is on an IPv6-only network:

![Joe-CLAT](/joe-CLAT.png)

**Conclusion**

It is here, the cell providers already use it, you can use it too. IPv6-only networks that is. Totally up to you if it is worth it, or even interesting for your specific user cases. But take note, change is coming:  
I myself have clients with branch offices in Asia that will lose their IPv4 connectivity during 2024. Yes, this is true, ISP:s in primarily China and India is actually dropping IPv4 to their paying customers.  
If you want IPv4 connectivity there, you'll have to tunnel over IPv6.  

During my 15 months of testing, I have tried many operating systems: - Linux, FreeBSD, iOS, OSX, Windows, Android/Chromebook. It really is a pity Windows doesn't work with CLAT. But please note: For clients that doesn't need the emulated IPv4 connectivity that CLAT/464XLAT gives - It will work just fine with NAT64/DNS64.  

Regardless of your own feelings regarding IPv6 - It is here to stay, you will have to deal with it sooner or later. Be the master of your own fate. Besides that, it's actually pretty fun.  


**Nifty Links if you want to know more:**

Check out IPv6 Buzz over at Packet Pushers. It is an amazing podcast featuring Tom Coffeen (yes, the very author of the book below), Scott Hogg, and Ed Horley - Three of the most knowledgeable people in the world regarding IPv6.  
There is a brand new episode that is all about exactly the above, check it out:  
[IPv6 Buzz: IPv6 CLAT and IPv6-only networks](https://packetpushers.net/podcast/ipb140-ipv6-clat-and-ipv6-only-networks/)  

The _only_ book you need regarding IPv6 addressing: [_IPv6 Address Planning_, by Tom Coffeen](https://www.amazon.com/IPv6-Address-Planning-Designing-Future/dp/1491902760)  

[NAT64/DNS64 in a Fortigate](https://docs.fortinet.com/document/fortigate/7.4.1/administration-guide/443324/nat64-policy-and-dns64-dns-proxy)  

[464XLAT](https://tools.ietf.org/html/rfc6877)  

[NAT64/DNS64 deployment](https://datatracker.ietf.org/doc/html/rfc8683)

[Migration strategies for Mobile Networks](https://www.apnic.net/wp-content/uploads/2017/01/IPv6_Migration_Strategies_for_Mobile_Networks_Whitepaper.pdf)

[ipv4only.arpa](https://www.rfc-editor.org/info/rfc7050)

[PREF64](https://datatracker.ietf.org/doc/html/rfc8781)

[Deploying IPv6-only networks](https://labs.ripe.net/author/ondrej_caletka_1/deploying-ipv6-mostly-access-networks/)  

[Tundra NAT64/DNS64](https://github.com/vitlabuda/tundra-nat64)

[The Network Undertaker - Start page]({{ '/' | absolute_url }})