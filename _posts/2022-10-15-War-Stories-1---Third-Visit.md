**Part 3**  

Same thing - A few months passed. Autumn turned into winter. This time the surprise call wasn't made late on a friday, but mid week.  
More consultants from other firms were involved now we were told. Firewalls needed to be configured for remote access. The work with setting up all networks must continue. Now we had gotten some documentation about the network design, and a working IP-plan with unique networks. So the objectives was much clearer now. We decided that I should go alone this time, my collegue was very busy at the time, and I was the one certified on the firewalls in use anyway, so...  

**Let's rock!**

![flight](/flight.jpg)  

Same flight, same hotel. Arrived early on monday. Met up with another consultant. We had an impromptu catch-up meeting in the hotel over coffee. It turned out he had been there quite a few times during the autumn (this was later on in winter). He also hadn't been able to be of much use during his earlier visits - He went through the same struggle with accessing the site, same struggle with not understanding why he was there, and not much information was given to him either. So every time, he had also been booked for a 2-day visit. And every time, he had been able to work just a few hours, if any. He also never actually met the guy that was supposed to be his guide and local contact. The very same guy eluded us both, and never showed up! So, the other consultant had a very similar story to what we experienced during our earlier visits. Anyway, we're consultants, another day, another dollar. It was time to work, so we headed for the DC on site.  

We arrived,  and was told by the guy that let us in that people were very disappointed with the network. We asked for more details, but he couldn't elaborate - He had just overheard this when being on site. Not being an IT guy himself, he couldn't give us any more information.   

We made ourselves comfortable (well, as comfortable as you can get in a DC), set up our computers on a table, and connected ourselves to the management network. Everything was unreachable. No answer from the gateway. No ARP. Nothing but a blinking switch port. We brought out console cables and connected to the main core chassis. CPU was at 100%, in a chassis capable of routing hundreds of gigabits per second. We had a really big loop, and it took a while to find it since the core was pretty unresponsive, for obvious reasons: Someone (not us) had tried to set up Ethernet Ring Protection instead of RSTP for the rings of switches that was placed around the really big railway yard, but without terminating the rings correctly. Basically creating a 10 gigabit loop per ring, then carrying on, creating another 10 gigabit loop in the next ring. Obviously not noticing, or not giving a rat's ass about the mess he created...  

We had to pull quite a few fibers in order to get the CPU in the core routers to go down so much that they were manageble, then we went on trying to sort out the ERP situation. It took us until lunch to get the core network back into a stable, running state with all links active. The other consultant then continued with setting up networks, and I started with my main task - Configuring one of the firewalls for remote access. The site was supposed to be completely air-gapped when up and running, but for now, the project management decided it was impractical to not have any internet access/remote access. That took a few more hours to set up, and with that done and tested, we decided to call it a day. On our way out, we got called out by a couple of guys standing at the entrance:  

"Hey, did you guys do something?" they said to us.  
"Yeah, we found a loop in the network. Well, actually several loops that blocked out just about everything." I said.
"Damn nice! We can reach our gear now. We haven't been able to for TWO F****NG WEEKS! Kudos, thanks guys!"  

>It is always nice to make a difference, at least sometimes!  

During our breakfast at the hotel the next morning, the other consultant was called by his boss who wanted an update about the progress. It seemed he was obligated to report back to the project managers in Italy how everything went, so he wanted to check in on us. The other consultant told what we had to deal with during the morning - Everything unreachable when we arrived, people had complained about the network, we found bad config, and had to pull fibers and reconfigure a lot of switches in order to get the core network back up. Boss thanked for the update, wished us a nice day, and hung up. We continued our breakfast, but was interrupted by another call - This time, it was someone from Italy, yelling that we need to have an emergency meeting NOW! Finish your breakfast, go somewhere quiet, call back and put the phone on speaker. NOW!  

We rushed to finish, and went back to the other consultants hotel room, called back and turned on the speaker.  Three guys at the other end were very angry. They didn't have time introduce themselves, they instantly went for our throats:  

"WHAT ARE YOU DOING?!" someone shouted on the phone.  
"We found a loop, the whole core network was down when we arrived. We had to start with some troubleshooting." the other consultant said.  
"WHY ARE YOU NOT WORKING ACCORDING TO THE PLAN?! YOU NEED TO FOLLOW THE PLAN!" someone shouted.  
"We couldn't work, in fact, nobody could work, since the core network was down. The server guys haven't been able to reach their systems for some time." the other consultant said, trying to explain.  
"YOU ARE NOT THERE TO FIX STUFF! YOU ARE THERE TO WORK ACCORDING TO THE PLAN!" someone else shouted over the phone line.  
"Well.." I started, and paused, getting a bit nervous now with all this shouting.  
"We had to do what we did, in order to be able to connect the firewall to internet and the management network, and then configure it for remote access." I continued.  
"YOU DID WHAT?!" someone shouted at the other end.  
"WHY THE HELL AREN'T YOU WORKING ACCORDING TO THE PLAN?! WHY ARE YOU DOING THINGS NOONE ASKED FOR? YOU ARE TO IMMIDIATLY STOP WITH WHAT YOU ARE DOING, AND CONTINUE TO WORK ACCORDING TO THE PLAN!"  
"AND BY THE WAY: Are you done with the configuration of the remote access? We need to get in."  

Almost like a flick of a switch, the tone in the call changed to normal voice. I glanced at the other consultant. He stared out into nothing, eyes wide open, mouth wide open. His gasket was about to blow, and boy, did it blow:  

 "ARE YOU OUT OF YOUR F****NG MINDS? You have spent ten minutes yelling at us for fixing your core network wich was looping like craxy! Then you continued to yell at us for doing what we were called here to do - Setting up remote access. Then you have the nerve to ask if we are done so you can access?!"  

 The yelling continued back and forth for a few minutes, but I kept quiet and more or less zoned out. Stared out the window. Usually, I also tend to get upset in these kinds of situations, but this one was too far off on a tangent for me to be bothered. I'm a tiny piece in this puzzle, no one on the other side of this phone have the slightest idea of who I am, and they don't care anyway. So why should I be upset with this? I have done my work, and I did it well.  

 The call ended in a more friendly tone when everybody finally calmed down - It turned out this is NOT how you do things in Italy - You are not allowed to fix anything without acknowledgement from a project manager. You are to work without questioning anything, and to never do anything on your own initiative. Figures. So, if you run into something unexpected, don't ever fix - Call your manager. Quite a clash from what we were used to.

 **Epilogue**  

 This was my last visit to this place. I have no clue how anything went further on. Don't care really either. I delivered what I was hired for. It is a great story to tell to get a few laughs. People find it hilarious, and don't believe what they are hearing. Many talk and think derogatory about "Consultants" - Consultants cost a lot of money, and they don't deliver much. In this story, this is painfully true. I maybe worked 8h in total during the 6 days I was there. Was that my fault in any way? Nope.  

 ![Cog](/cog.jpg)  


 In this case, we were tiny cogs in a huge machinery (again, with the cogs...). The total budget for this entire project was several billions of dollar. Thousands of people involved. It is not an easy feat to make sure every resource is in the right place at the right time, all the time. But flying in network techs from another country when you should have called an industrial electrician to fix the mains is a mistake that can be easily avoided. Creating an IP-plan that isn't complete hogwash is also a good start if you are a network architect. In the end - This inefficiency on several levels must have costed huge amounts of money during this project's entire lifespan. Tax payer's money. As much as I like getting payed well, I really don't like being a part of something so wasteful.
 
 This project learned me a lot about how different the work culture can be can be in different countries. I am used to a culture where you have a lot of freedom, within certain boundaries of course. Creativity is encouraged. You are trusted with solving things on your own if possible, without going too much "gung-ho". Besides that, I don't have a manager to call for permission anyway, so...


[Go to the Home Page]({{ '/' | absolute_url }})