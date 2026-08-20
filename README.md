# rpi0w-revival
This might not be the best place for instructions on how to setup a raspberry pi zero W to operate as kiosk displaying a webpage in 2026 but what else am I going to do?  Start paying someone to host a webpage for me?

## Rant
Single board computer prices are wildly increasing right now.  The former fame of the $35 raspberry pi is gone.  On august 16th 2026 it was $94 for a rpi 4b with 2GB memory, $209 for a rpi 5 with 8GB RAM and $399 for rpi 5 with 16GB RAM starter kit or $305 for the board by itself.  Rpi zero 2W is only $17... HAHA jokes on me... it's out of stock.  If I spend $89 I can get it at Canna as part of a kit with a stupid case, heat sink, and power supply.   Hmm no thanks to all of the above.<br>

I remember when I could spend somewhere around $20 and get a pi zero w then spend $20 more for any cables or maybe a power supply and try out a new os or new software project without taking out a loan or dipping into savings (I'm kidding).<br>

Fast forward to the age where even I can setup home lab software (self depricating but don't worry I have an ego too to keep in check).  I setup Home Assistant on my Rpi 5 and I like it... but now I want my Pi 5 back and really won't buy a replacement with these crazy prices.  So I have the Home Assistant project running on my RPi 3B+ (barely but it works).  Lights and power outlets can be turned on and off using matter, thread, and the ZBT-1 hardware for a thread border router. I even jumpered a capacitor to 5V and Gnd pins so to get rid of the constant under voltage or brownout problem.  Thanks Taulab project for making me learn about brownouts and under engineered power paths!<br>

Now I want to setup a Magic Mirror display (at first I thought of Dakboard, but why not self host MM2 instead).  I have two Pi Zero Ws left and I thought this HW should be sufficient.  Little did I know software bloat crept in, mainly with the browser, while I wasn't looking.  I tried using the latest 2025 Raspbian Trixie with chromium and then Firefox-esr and both failed to run due to a missing instruction set that doesn't exist on the Arm6 in the RPi Zero W.

## Problem Statement
New Raspberry Pis are heavily affected by the memory cost increase affecting all computers currently in 2026 and they are no longer a good value.  I'll be using older raspberry pi zero W hardware to build a simple Magic Mirror 2 display or possibly a Dakboard display.  The issue with using older hardware is modern web browser version will not run.

## Progress
I setup the display in two different ways.  One would load and display but then crash after hours of operation.  I'm trying a different method now.  The software stack is <br>
- 32-bit DietPi
- DRM/KMS
- Graphic packages with Wayland support
- Cage
- Cog
- WPE Webkit 
...to be continued ...
