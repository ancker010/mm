---
published: false
layout: posts
comments: false
title: Conditional ISP Failover with Mikrotik/RouterOS
---
## Conditional ISP Failover with Mikrotik/RouterOS

Preface: I've been long-time subscriber to home automation. It started with the original Nest thermostat. I loved being able to control it via an app on my phone. The 'learning' part wasn't as interesting since shortly after installing it, I switched to 100% remote work.

Not too long after the Nest, I was introduced to the Staples Connect hub. It had Z-Wave and Zigbee radios and also an app for my phone. I quickly installed a handful of z-wave door sensors and got a z-wave garage door controller. Getting alerts on opened doors and being able to control the garage door from afar hooked me. Everything must be automated.

Staples Connect was eventually sunset so I moved to SmartThings. SmartThings started getting unreliable (too much reliance on Cloud for what should be local rules), and the subsequent purchase by Samsung. The delayed alerts and actions (sometimes 30+ minutes) were enough to push me to find other options. The stand-out option was Hubitat Elevation. I looked at home-assistant and a few more, but Hubitat had the greatest amount of community supported apps and devices combined with not having to spend too much time in code. I've now moved to 100% Hubitat, which means all of the home automation actions are processes and executed locally. This is huge. Everything works (and quickly) even if our ISP is down. 

So how did all of this lead to this blog post? **Alerts**

While our ISP is generally pretty stable, I didn't like the idea of missing an alert or being unable to check the status of the house while away from home due to an ISP outage. 
