---
published: true
layout: posts
comments: false
title: Conditional ISP(Cellular) Failover with Mikrotik/RouterOS
categories:
  - blog
tags:
  - networking
  - Home-Automation
  - Hubitat
  - Cellular
  - Mikrotik
---
## Conditional ISP Failover with Mikrotik/RouterOS

### Preface
I've been long-time subscriber to home automation. It started with the original Nest thermostat. I loved being able to control it via an app on my phone. The 'learning' part wasn't as interesting since shortly after installing it, I switched to 100% remote work.

Not too long after the Nest, I was introduced to the Staples Connect hub. It had Z-Wave and Zigbee radios and also an app for my phone. I quickly installed a handful of z-wave door sensors and got a z-wave garage door controller. Getting alerts on opened doors and being able to control the garage door from afar hooked me. Everything must be automated.

Staples Connect was eventually sunset so I moved to SmartThings. SmartThings started getting unreliable (too much reliance on Cloud for what should be local rules), and the subsequent purchase by Samsung. The delayed alerts and actions (sometimes 30+ minutes) were enough to push me to find other options. The stand-out option was Hubitat Elevation. I looked at home-assistant and a few more, but Hubitat had the greatest amount of community supported apps and devices combined with not having to spend too much time in code. I've now moved to 100% Hubitat, which means all of the home automation actions are processes and executed locally. This is huge. Everything works (and quickly) even if our ISP is down. 

So how did all of this lead to this blog post? **Alerts**

While our ISP is generally pretty stable, I didn't like the idea of missing an alert or being unable to check the status of the house while away from home due to an ISP outage. I wanted a way to switch over to a secondary ISP (Cellular in this case) only when the main ISP has an outage.

Let's get started!

### Assumptions
In order for this to work, a handful of things need to be in place.
1. Your main home router needs to be a recent Mikrotik capable of running 6.x. (Tested on 6.47)
 - I'm using a Mikrotik hEX.
2. No special routing config, VLANs, or existing policies.
 - You can likely make it work, but these won't work without modifications.
3. The secondary ISP modem operates in a "**Router**" mode, and has a "**DMZ**" or "**Forward All**" mode to a specific client.
 - My Cellular/LTE modem provides a subnet 192.168.5.0/24 for clients. I've statically assigned 192.168.5.10 to the "lte" interface on my Mikrotik, and set this IP in the modem DMZ settings.
 - Some Cellular modems provide a "**Passthrough/Bridge**" option where the cellular IP gets assigned directly to the client. I couldn't get that to work reliably, so I chose to leave the modem in "**Router**" mode. If I decide to work more on that, I'll come back and add an addendum.
 - For reference: I'm using a Netgear LB1120 LTE Modem.
 
### ISP/Cellular Setup
 
The first thing you need to do is get a secondary ISP or figure out your cellular plan. While researching cellular options for another project, I came across a [pre-paid SIM card](https://www.embeddedworks.net/wsim4827/) that offers unlimited data at 64kbps on T-Mobile, AND provides a static IP option. Static IP isn't required, but it means I can add it to my smokeping config pretty easily. Of course, you need to do some research to ensure that whatever cellular plan you decide to go with is supported by whatever Cellular Modem you plan to purchase. The Netgear LB1120 seems to support most of the common bands in the US/Canada. You'll need to verify for your region and area.

Once you have your SIM and modem in hand, obviously you'll need to verify everything works. I won't go into how you should do that, but I essentially connected it to my laptop and made sure I could browse the internet via cellular. It worked surprisingly well for a 64kbps connection speed.

Once you've verified it works and passes traffic. Plug the client part into your Mikrotik. I chose ether5.

### Mikrotik Set Up

First, you'll need to set up/name the interface something useful. I chose '**lte**', not to be confused with '**lteN**' interfaces that might get created if you plug an LTE modem into the USB port on some Mikrotik devices.

NOTE: You can do 100% of this via the Winbox/Webfig interface. It's just WAY easier to provide CLI commands instead of million screenshots. You can probably figure out where to look in the GUI tools by reading the commands...

```
interface ethernet
set [ find default-name=ether5 ] l2mtu=1598 mac-address=00:0D:B9:aa:bb:cc name=lte speed=100Mbps
```

Next you'll need to set up a static default route towards the LTE modem. There are two important bits to pay attention to here.
1. Set the distance to something higher than your primary ISP. (I chose 5)
2. The routing-mark will be used later on. Make sure you set it to something that makes sense. (I chose use-lte).

```
ip route
add comment="Default Route to LTE" distance=5 gateway=192.168.5.1 routing-mark=use-lte
```

Next, we'll set up some firewall NAT and Mangle rules to massage the traffic to our will.

To start, we'll need to set up NAT for packets egressing the **lte** interface.
```
ip firewall nat
add action=masquerade chain=srcnat out-interface=lte
```

Next a handful of Mange rules are necessary to mark connections and packets to ensure that when packets are received on the **lte** interface, they egress back out the **lte** interface. 

This was a little confusing me at first, so I'll try to explain what they do in order.
1. Any packets received on the **lte** interface, set a connection-mark of **lte-packets**.
2. Any packets that are forwarded by the router that have a connection-mark of **lte-packets**, set the routing-mark **use-lte**.
3. Any packets originated by the router itself and sent out the **lte** interface, set the routing-mark **use-lte**.

```
ip firewall mangle
add action=mark-connection chain=prerouting in-interface=lte new-connection-mark=lte-packets passthrough=no
add action=mark-routing chain=prerouting connection-mark=lte-packets new-routing-mark=use-lte passthrough=no
add action=mark-routing chain=output connection-mark=lte-packets new-routing-mark=use-lte passthrough=no
```

Last, we need to set up a specific rule to set the **use-lte** routing-mark on packets from my Hubitat hub. BUT, we'll want to keep it disabled so it only gets used when the primary ISP is down.

```
add action=mark-routing chain=prerouting comment=from-hubitat-rule disabled=yes new-routing-mark=use-lte passthrough=no src-address=10.100.100.82
```

And last-last, we should probably add some firewall filters to prevent nefarious things from happening. 
1. We allow inbound **ICMP** (so things like smokeping pointed at the cellular IP will work).
2. We allow only **ESTABLISHED** and **RELATED** packets to ingress the **lte** interface. 
3. And then we drop everything else.

```
/ip firewall filter
add action=accept chain=input connection-state=established,related in-interface=lte protocol=icmp
add action=accept chain=forward connection-state=established,related in-interface=lte
add action=drop chain=input in-interface=lte
```

### Testing time!

Everything we've done so far is all you really need to allow your chosen device to use the cellular modem. You can manually enable that last rule and see if your device can ping/talk via cellular. Go ahead and try it!

**NOTE:** If you are trying to trigger a push notification from a Hubitat hub, you'll likely find that it doesn't work. The reason is that the Hubitat hub maintains a constant TCP session with a host in AWS for push notifications.

Okay..what's that mean. Well, Mikrotik/RouterOS does connection tracking for NAT. By default the timeout for **TCP Established** is set to 1 day. WAY longer than I want to wait for things to fail over in an outage (and frankly longer than I've ever had an ISP outage). You can change this to something lower with the following. There may be rammifications for setting this too low, but likely nothing too worrisome.

```
ip firewall connection tracking
set tcp-established-timeout=2m
```
Or if you don't want to wait...
```
ip firewall connection print
... find all of the connections for your chosen device ...
ip firewall connection remove <numbers>
... <numbers> represents the list of connections you found.
```

### So how do we make it happen automatically?

Mikrotik has a built-in function called a **netwatch**. It's not very eloquent, but it allows you to run commands (or scripts) based on whether a given IP is reachable via ping. We'll set up two netwatch rules below.

**NOTE:** There are likely better ways to do this, like with scripts or some other method. This worked for me and has proved to be pretty simple to get up and running.

First you need to pick an IP on the general internet that should pretty much always be reachable. I chose Google's public resolver **8.8.8.8**. (This IP probably gets Gbps of ICMP traffic...) 

Then we create a netwatch rule. There a few interesting bits here, so I'll try to explain.
The rule has three components. 
1. Target host (and how own to test, I chose **8.8.8.8** and every **30s**).
2. A **down-script**. (A list of commands to run when 8.8.8.8 is unreachable. Primary ISP Down)
3. An **up-script**. (A list of commands to run when 8.8.8.8 is back. Primary ISP Up)

Down-script, unpacked.
1. Enable the rule for packets from the Hubitat hub we set up (but disabled) earlier.
 - `:set [/ip firewall mangle set [ find comment=\"from-hubitat-rule\" ] disabled=no];`

Up-script, unpacked. 
1. Re-Disable the rule for packets from the Hubitat hub. (Remember, we only want it enabled when the primary ISP is down.)
 - `:set [/ip firewall mangle set [ find comment=\"from-hubitat-rule\" ] disabled=yes];`
2. Clear/Remove all connections that are currently marked with the lte-packets connection-mark. (We do this to make the Hubitat hub immediately establish a new connection with the notification server back on the Primary ISP.)
 - `/ip firewall connection remove [ find connection-mark=lte-packets ]`

```
tool netwatch
add comment="Check for primary ISP function (ping 8.8.8.8). Disable/Enable Hubitat table/rules based on result." down-script=\
    ":set [/ip firewall mangle set [ find comment=\"from-hubitat-rule\" ] disabled=no];" host=8.8.8.8 interval=30s up-script=\
    ":set [/ip firewall mangle set [ find comment=\"from-hubitat-rule\" ] disabled=yes];\
    \n/ip firewall connection remove [ find connection-mark=lte-packets ]"
```

The clever among you might say "But Ryan, since we picked 8.8.8.8, wouldn't that be reachable via Cellular as well?" Yes, it would. So we create a firewall filter to block it on the **lte** interface. 

```
ip firewall filter
add action=drop chain=input comment="Drop inbound from 8.8.8.8 on lte for netwatch script" in-interface=lte src-address=8.8.8.8
```

And if you want, (this part isn't required), I set up a second netwatch rule to check the status of the cellular modem itself. It can definitely be improved as I'm just checking whether the modem itself responds, NOT whether the cellular service is working. If it's down for whatever reason, there's no reason for the first netwatch rule to try and alter anything. I set it up as follows.

```
tool netwatch
add comment="Disable the 8.8.8.8 check if LTE is dead." down-script=":set [/tool netwatch set [ find host=8.8.8.8 ] disabled=yes]];" host=192.168.5.1 interval=15s up-script=\
    ":set [/tool netwatch set [ find host=8.8.8.8 ] disabled=no]];"
```

### Testing Time 2!

Everything is in place for automatic failover (and back) if your primary ISP fails. But how do we test it without unplugging primary ISP? 

Simple. You add a static route to point your chosen test host (8.8.8.8 in my case) to the lte interface, where we have it blocked. And since we have a netwatch rule for 8.8.8.8, it will appear down and trigger our failover. Here's how I built it. It's disabled by default.

```
ip route
add disabled=yes distance=1 dst-address=8.8.8.8/32 gateway=192.168.5.1
```
Any time you want to test your failover, you can toggle this static route and traffic from your chosen host (my Hubitat) switch to the cellular connection and back.

### Conclusion
There you have it. A functional automatic failover system. Is it perfect? No. But it will get the job done 90%+ of the time.

Feel free to drop me a line on twitter if you have any questions.