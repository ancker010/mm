---
published: false
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