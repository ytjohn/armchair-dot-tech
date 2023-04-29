---

title: Enable linux serial console
author: ytjohn
date: 2018-01-19 12:40:36
tags: [yourtech-dailies]
tags: [serial]
slug: enable-linux-serial-console

---
## Serial port console

This is how to get serial port console working on a Ubuntu 16.04 (or any systemd based OS) and how to access it with idrac/ssh.


To get serial port working on a running system:

```bash
systemctl enable serial-getty@ttyS0.service
systemctl start serial-getty@ttyS0.service
```

**Update Grub:**

To get serial console during boot up, including the grub menu:

Go ahead and edit `/etc/default/grub` 

```bash
GRUB_CMDLINE_LINUX_DEFAULT="splash quiet"
GRUB_CMDLINE_LINUX="console=tty0"
GRUB_TERMINAL="console serial"
# also, it takes so long to boot a server, adding 10
# second to the grub menu is more good than harm
GRUB_TIMEOUT=10
```

**Access it via idrac**

```bash
ssh <idrac-ip> console com2
```


#yourtech-dailies
