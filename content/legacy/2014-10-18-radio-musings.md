---

title: Radio Musings
author: ytjohn
date: 2014-10-18 17:58:11

layout: post

slug: radio-musings

post-date:
  - 'a:1:{i:0;s:37:"a:1:{i:0;s:19:"2014-10-17 22:25:21";}";}'
---
Completely unrelated to the post below, I discovered that some has [forked a copy of my website](https://github.com/builtdock/ytwebsite) on github. I've actually migrated from the static site to [Mezzanine](http://mezzanine.jupo.org/), but the repo is still in place. I was very surprised someone has forked it. I checked the user out and he recently forked a large number of repositories, all related to docker or coreos type things. It appears that "builtdock" is putting together some sort of platform as a service offering. I wonder if forking my site was an accident, or if they are planning to use it as a base of making their own static site.


----------


I recently took my Raspberry Pi, connected a GPS, installed Xastir and have that setup as a sort of tactical display in my office. [<img src="https://lh3.googleusercontent.com/--RicSliZ2kE/VDb9oB4auGI/AAAAAAAAT1M/eCkTaa8AIEg/w876-h1168-no/14%2B-%2B1" height="200" width="149">](https://lh3.googleusercontent.com/--RicSliZ2kE/VDb9oB4auGI/AAAAAAAAT1M/eCkTaa8AIEg/w876-h1168-no/14%2B-%2B1 target="_new").

I have some interesting things I came across recently.

The first interesting project is from hackaday, a 0-30Mhz Portable SDR with transceive capabilities, opens-source design. It'd be interesting do this, but use a Pi+Touchscreen for the CPU portion.

http://hackaday.io/project/1538-portablesdr

The second one has been around for a while, and it's called [Earl](http://www.meetearl.com/). This is a crowd-funded, e-ink touchscreen android system with gps, built-in maps, and a VHF/UHF transceiver for GMRS/FRS/2m/70cm walkie talkie type capabilities. The radios are software defined, so something like DroidAPRS would be able to talk to the radio directly (perhaps with a bit of coding). I've been keeping and eye on this project (because I want one), but never thought it would actually materialize due to FCC rules concerning GMRS and FRS type radios. However they are now actively working through the FCC approval process and modifying the radio portions to get acceptance. I am hoping that enough get produced that some of these end up on ebay by non-hams that are disappointed in the radio communications aspect.
