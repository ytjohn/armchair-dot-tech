---

title: Programming Uniden AMH-350 for APRS
author: ytjohn
date: 2015-08-03 17:58:11

layout: post

slug: programming-uniden-amh-350-for-aprs-2

---
This is a narrative post. If you want to see my python program that calculates out the diode matrix, skip to the end or [click here](http://nbviewer.ipython.org/gist/ytjohn/719504772237953eb28d),

I recently received this "[Force Communications AMH-350](http://i.imgur.com/mFzoKU5.jpg)" radio. Actually, it was an entire cabinet with a large power supply, an MFJ TNC2 tnc, and an old DOS PC running JNOS. These had active in a tower shed and turned off 3 years ago. The club wanted me to repurpose this packet system for APRS.

Once I plugged it in, the computer booted up to JNOS, but the radio and TNC did not turn on. The power supply had a plastic box on the back with a larger bussman 30A fuse. When I pulled it out, corrosion dust leaked out. I made a trip to the hardware store and replaced it. The radio turned on but not the TNC. On the front I found 3 smaller fuses and a note describing that "F3" ran the TNC. Pulled that fuse out and it was dead. A second trip to the hardware store got this fuse replaced. Then I plugged everything back in and turned on the power supply. Within 10 seconds, the "make it work smoke" had leaked out of the TNC2. This is probably why the F3 fuse had blown in the first place. This was disappointing, because there is new firmware for the TNC2 that makes it a decent APRS TNC, no computer needed.

The computer, I deemed too old to run a soundcard packet (using direwolf as my driver), so this left me with the power supply and radio. Grounding out the PTT line and using a frequency counter, it showed me "channel 2" was transmitting on 145.050. Channel 1 was not programmed at all.

A quick google search told me that Uniden bought Force Communications and sold this radio as a Uniden AMH-350. I found 2 other people looking for how to program it (one in 1994, and the other in 2004) with no response. I found someone selling the radio's manual on ebay for $20. I offered them $10 and received the manual earlier this week.

The radio itself is programmed with a common cathode diode matrix, representing a binary value. Here is a picture of one [back side of it](https://lh3.googleusercontent.com/PU3U5GAr6WoglosW7O8Vo0x_cdAc6eTc_Zt2UHmYooda=w860-h1526-no) programmed for 145.050. The manual provides a table covering frequencies from 148Mhz to 174Mhz in 5khz increments. Fortunately, it provides a formula on how to come up with your own frequencies. I ran through this formula multiple times getting different results from the book, till I realized the book was rounding some values UP or outright disregarding fractional parts. It also took a bit to wrap my head around binary "1" being disconnected (or cut) and binary "0" being connected. That felt backwards to me.

Eventually though, I was able to match the book, create a chart that matched the existing programmed 145.050 frequency (both Tx and Rx, which are programmed separately). Then, I wrapped the whole thing up in a set of python functions inside an ipython notebook. You can view this on ipython's [nbviewer](http://nbviewer.ipython.org/gist/ytjohn/719504772237953eb28d) or the direct [gist](https://gist.github.com/ytjohn/719504772237953eb28d).

I don't have the radio programmed yet. I feel getting the diode matrixes out of "channel 2" and still having them useful for programming with is going to be difficult. I will need 7 diodes connected for each Tx and Rx slot, 14 total. I am attempting to program up channel 1. By the time I got to this portion, I was a bit tired and making mistakes, so I called it a night.  Once I get to building out the programming board, I'll post some more pictures.
