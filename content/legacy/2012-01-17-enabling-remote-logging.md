---

title: Enabling remote logging
author: ytjohn
date: 2012-01-17 11:11:20

layout: post

slug: enabling-remote-logging

---
I was experimenting with rsyslog to allow remote logging and within 20 minutes, I had two rogue log entries show up from random hosts.

Jan 17 05:00:23 64-181-24-18.unassigned.ntelos.net kernel: Configuring keys for mac:14:5a:05:ad:6b:cd, sc_hasclrkey is set
Jan 17 05:01:05 64-181-24-18.unassigned.ntelos.net kernel: Configuring keys for mac:14:5a:05:ad:6b:cd, sc_hasclrkey is set

So what is interesting about this? I'm on comcast in PA. The host that sent this is on ntelos.net, or Lumos in (most likely) West Virginia. For some reason, a system there is sending a log message to my IP address. I just now am accepting it.
