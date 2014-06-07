OS X Server 
----------

Let's do a step-by step process of how I setup my server. I'm mostly using the server to access my computer with screen share (fast than many clients I tried: Jump Desktop, VNC clients) as well as running continuous integration through Apple's ```OS X Server```. Many developers I have talked to have just began using this new feature (as of Mavericks), and I'm excited to see how it pans out as well. In the meantime, let's go through how I integrated OS X server and xcode to run my apps.

If you're interested, I'd check out what purpose Unit Testing and Integration testing serves in software development. iOS has a lot of books on the subject, Ray Wenderlich has great tutorials, as well as objc.io. Ideas of what the difference of those two are? (hint the names give a big hint). For a quick note about the reasons WHY testing is great, think about having to deploy your system several hundred times a day by hand just to see if the server calls don't cause an app crash, persistent state consistency, or even checking it under brute force users. Even shorter, it tries to find the errors before the users do. 

What's the risk not using testing? Lot's of single stars for your app. Now, many developers test using their own hands and for the most part that is sufficient for a handful of users that may download your app. But what happens if your app all of a sudden gets really popular, and you're API calls now sky rocket? What happens when you exceed your limit of the day at 9am - and you lose hundreds or thousands of more downloads? There comes the pain! Enough of why testing is useful, let's set up OS X server!

First, if you're a developer (and there's really no reason not to be) you can download it for free from your dev portal. Otherwise, you can buy it for a moderate price from the mac app store. When you do download the server, it will ask you some fairly boilerplate (typical) questions about your settings. Nothing too fancy there, just fill out what you think you may need, and you can always change things later as well.

Next you'll see a great looking taskbar with a bunch of icons to the left, and we're looking for xcode:

```
IMG HERE
```

Here you will want to turn on xcode, and after it runs for a little while, you'll be able to launch your testing interface with your given hostname. Voila, you're ready for continuous integration!

###Setting up to a domain
Now, what if you want to access it externally? If you have a domain name, all you need to do is link your home IP address to that domain name and you will be able to access your router (we'll cover port forwarding in a second). First, a little about IP addresses... your IP address is serviced to you by your internet service provider (for me it's Comcast). Your service provider will ever so often change your IP address on you, and you will have to take this into account if you ever do link a domain or try to access your router from the WWW. Many static IP websites can manage this for you by downloading their service and having them host an IP that is universal while forwarding it to your computer whenever your IP changes. Voila, that's a fix. For now, let's assume you'll manually have to change it and move forward. P.S. if you don't know your public IP address, type my IP address in Google and it should appear!

Now, getting back to port forwarding... if you enabled Xcode as a service from the start, you actually get these settings for free if you're on an Apple network switch (wireless router). But let's assume you do have an Apple network switch and need to set it up manually. This process is also true for other wireless routers (linksys, etc), so still follow along if you don't have an apple product.


Open up Airport Utility or whichever utility to access the network switch and gain access to your base station. Here you will navigate to a network or advanced section where you will look for the terms: "Port Forwarding" or "Port Settings." You may want to keep your computer as a certain DHCP address, which is done also on your network utility admin as well as your personal computer. Next for port forwarding, we are looking to add an IPv4 setting, and feel free to call it Xcode. You can use the following boilerplate settings that apple suggest:

```
Public TCP Ports: 80, 443, 2012
Private IP Address : 10.0.1.XX or 192.168.0.XX
Private TCP Ports: 80, 442,2012.

You can have some change in these ports, but take a look at Apple's port settings to see ones you don't conflict with!

Hit save and allow time for your network switch to reset. Next you should be able to go to your IP address, and your wireless router will then forward straight to your machine running Xcode! Cool huh?

You can also set up websites and other network services through this port forwarding so enjoy!

Cheers!