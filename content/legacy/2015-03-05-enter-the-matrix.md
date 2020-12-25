---

title: Enter the Matrix
author: ytjohn
date: 2015-03-05 17:58:09

layout: post

slug: enter-the-matrix

1:
  - 'a:1:{i:0;s:7:"=======";}'
2:
  - 'a:1:{i:0;s:52:">>>>>>> rollback1:raw/2015-03-05-enter-the-matrix.md";}'
---
<p>I used to be a <a href="http://www.yourtech.us/blog/jabbering-about-python/">big proponent of xmpp</a>. However, over the years my enthusiasm has waned for it. I'm <a href="https://www.eff.org/deeplinks/2013/05/google-abandons-open-standards-instant-messaging">not the only one</a>.  Essentially, these days if your chat service is not done over HTTP(s), and if it doesn't have persistence, your chat service is now legacy. Yes, I still enjoy IRC, and I think it's great for ephemeral communications. But in this multi-device, mobile world, it's hard to use IRC as a daily driver for my friends and coworkers. </p>
<p>Several months ago, I started looking into chat systems again for a different reason than most - amateur radio. There's this thing in amateur radio called <a href="http://www.broadband-hamnet.org/">Broadband-Hamnet</a>, which is a wireless mesh network. It's not the first mesh system out there, but it has a really good initiative behind it.  The idea behind it is that all nodes are configured to use the same SSID and the network is self configuring. If I stand up a node here at my house, someone else, having never spoken to me before, could deploy a node within range of mine and the two would connect. They would be able to see the node, any services I offer, and use them. DNS and service advertising is built in. </p>
<p>I wanted to come up with some "generic" mesh nodes with a connected server (raspberry pi). The idea that you could grab a couple of these boxes, deploy them in the field and operators would be able to share files, chat, and even video. The big catch was that you never knew what systems would be online at any given time.</p>
<p>I looked into standing up an IRC server with a web front end. This had a problem in that no historical messages would be synchronized during a netjoin. There are a number of P2P chat systems, though most of these require some sort of "bootstrap" system. Even worse, for an amateur radio system under FCC regulation, most of these are focused around encryption. Tox.im would be a good choice, except it would violate the no message obscuring rule of FCC part 97 that governs the Amateur Radio service.</p>
<p>I even started conceiving of a system based on the idea of a pub/sub message queue, except json over http. Nodes would subscribe to a channel and any message posted to a channel would get propagated to all the subscribing nodes. Using twisted, I could also create gateways for standard IRC or XMPP clients. </p>
<p>Well fortunately for me (and you) a group went out and did just that, only much much better than anything I could have put together. <a href="http://www.matrix.org/">Matrix.org</a> has put together a federated chat specification. The concept is really simple - json over http(s).  They have a reference implementation called Synapse that is written in twisted. People run homeservers of synapse and will join channel. A channel is shared between all homeservers that subscribe to it and all channel events are propogated until consistency is achieved. This means that if a homeserver joins the channel late, or goes a way for a while, it will eventually achieve a complete history of all message events within the channel.</p>
<p>If you run your server on the default port of either 8008 for HTTP or 8448 of HTTPS, the only DNS record you need is an A record. If you use another port like 443, then you add a DNS SRV record stating the host and port (just like with XMPP).</p>
<p>While the project still has a few rough edges, it is definitely usable today. The most stable implementation is on <a href="http://www.matrix.org/beta">matrix.org</a> but you can also join my homeserver at <a href="https://matrix.ytnoc.net/">matrix.ytnoc.net</a>.</p>
