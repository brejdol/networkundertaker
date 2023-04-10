**Part 2**

Some time passed. A few months later, it is early autumn. Everybody is back after the summer vacation. Then my collegue gets another late friday evening surprise call. It was pretty much a carbon copy of the first call, ending with:  
"You need to be here on monday morning at 08:00 AM. Can you make it?"  

Yeah, we could make it. But we at this time we wondered if anyone else on the site could. Make. Anything?

**Let's rock!**

![flight](/flight.jpg)

Same flight, same hotel, almost the same procedure, minus the first day. Our guide from last time met up with us at the site gates, and welcomed us back, trying to smooth over the rough edges we experienced last visit.  

"It is a big project, it is hard to get every thing done in the correct order."  

Yeah, we knew the drill. This time though, we had been promised that the DC would be powered up and ready to go. This time we also had some more information on what kind of work we were supposed to do - Basically our task was to set up a lot of networks for each and every train station in the core. A spreadsheet was handed to us via a USB stick - The file was over 100Mb in size wich seemed odd when copying it outside the DC while waiting to get in. We didn't really have any time there and then to check it out.  

This time, there was power in the DC. And light. The mains were connected. We made our way through the building towards the central DC. We opened the door, and a very warm gush of air surprised us. Sauna warm air really. The first thing we see inside is a very bulky (like, really fat) and sweaty man in nothing but socks and briefs, sitting on a small chair with a laptop on his knee.  

He looked up, and shook his head towards us.  

"This is crazy!" he yelled over the datacenter noice, with a distinctive southern accent (Louisiana?).  
"They are forcin' me to get my systems online even though the f****ng cooling never worked!"  
"This shit will break within a few months!"  

At approx. 120F, the DC wasn't really a place you would hang for long. Nor place any kind of networking equipment. Nor servers.We did a quick check of the primary core - It was up and running, and had quite a bit of config in it. No errors. Yet. We decided to make a run for it, and go back to the hotel to get something cold to drink and then check the spreadsheet, and plan some of the work ahead.  

After cooling down for an hour or so in the hotel lounge, we opened our laptops and started to go through the spreadsheet. It was huge. Maybe a hundred different tabs. The logic was fairly easy to follow - One sheet for the train station's internal networks, one for the same station's tech networks (heating, air telemetry etc). Rinse and repeat. Same for the next station. And the next. Pretty clear. One thing that wasn't clear was the IP scheme. All network had the same gateway, and the same subnet. All of them! Every tab, every station, every internal network and every tech network. Had. The. Same. IP-addresses.  

>All networks was 10.0.0.0/24. Gateway on .1...

We had checked through the core router's config. It wasn't advanced to any degree at all, just a few management networks. A few VRFs. Some OSPF. No MPLS, no BGP. Nothing fancy. There was one VRF for all stations, and one for all stations tech networks. We were going to have a meeting with the project managers the next morning, wich was good, since we now had quite a few questions to ask them.  

The next day, we went to the central office for the meeting, after breakfast. We were greeted by some local guys, and the project managers attended on conference call from Italy. Meeting started with the project managers going through all the fiber cable maps. For the whole city, more or less. We listened for a while, then suggested an update to the meeting agenda:  
"If this meeting is to get us up to par with what should be done, we should focus on the work that should be done - We don't work with fibre, we don't really care where the cable ducts were dug down." We were met with complete silence.  

"It is extremely important that you know this, so that you can have this in your planning!" one of the project managers stated.
"Well" I replied. "We don't work with this, we don't handle any fiber connections. Why is it important for our work? Aren't we here to configure your routers?"  

Again silence. Then one of the project managers asked us to wait while they reasoned on their side. A few minutes passed. They came back:  
"Alright, we will fockus on the local networks between core and stations then!" one of the Italian project managers stated.  
We just had a few seconds to ponder upon the word "fockus" before it started. What started? Well, the endless stream of english mangled to something out of this world:  
"Let's fockus on foc three, shit five!" (FOC= Fiber Optic Connect, shit = sheet). A stack of blueprints over the local fiber connections was rolled out.  
"Yes, we have shit five out!" my collegue said, playing along with the diphthons so to speak. No one raised an eyebrow.  

After 90 minutes of shits and focks, we were exhausted, but we had to bring that IP-scheme up.  

"This spreadsheet... How are we supposed to execute it in the router? All subnets are the same, and there are no documentation on how this should be handled. It is possible to handle this, but that will take a lot of work, and for no real good reason." I said.
The reply was angry: "You are to work according to the planning, it has been done by the central networking team. They are experts! It isn't your work to question this!"
"Yes, and I will according to the planning. But after I type in the first IP-network, I will not be able to type in the exact same IPs in next network, or next, or next. How should this be handled? Please advice us." I replied.  

The conference call guys asked for a few minutes of talking privately. They came back after a while:  
"The IP-planning is a work in progress. More networks will be added later. You are to work according to the plan!".  
The meeting ended, without the two of us understanding much more. We went to the DC and added the first station network. Then we were done, since no more work could be done. This concluded our second visit to the site. And guess what? We had to go there a third time!

[War Stories 1 - Third Visit](https://networkundertaker.com/2022/10/15/War-Stories-1-Third-Visit.html)

[Start page]({{ '/' | absolute_url }})