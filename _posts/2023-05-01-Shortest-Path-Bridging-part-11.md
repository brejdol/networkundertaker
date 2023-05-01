**Summary**

Ok, that quickly turned into way more posts that I expected initially. The intent was to try to write down something that would discribe this technique fairly in depth, and explain how the move from a legacy environment to a full non-ip-based SPB-mesh complete with services can be done. 

There aren't that many operational guides on how to use SPB, nor which problems it actually helps to solve. And I think the protocol deserves better. 

As long as you understand SPB's boundaries, it is pretty amazing - You can do a lot with just a little configuration. You end up with a sturdy and resilient modern network that gives you a set of flexible tools that will help you to solve problems further ahead. 

If you add MACSEC encryption with AES256 on all links (which is supported in both ALE and Extreme switches), you will have a VERY secure transport network, covering everything up to the application layer and the data payload.

Still, SPB is a seriously niche technique - Supported by few vendors, used by even fewer users. That is the harsh reality. There are a few really big deployments out there in the wild though. None of which I have touched, and none of them will ever end up as examples in any blog posts unfortunately.

If you happen to have local networks using ALE and/or Extreme gear, there is really nothing to be afraid of - You have all to gain and nothing to lose. SPB ftw!

[Start page]({{ '/' | absolute_url }})