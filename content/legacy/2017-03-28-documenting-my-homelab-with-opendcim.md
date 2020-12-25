---

title: documenting my homelab with openDCIM
author: ytjohn
date: 2017-03-28 21:50:36

layout: post

slug: documenting-my-homelab-with-opendcim
tags:
- homelab
---
## Background

I've been wanting to document my home and homelab network for a while now. I used to keep some individual files around with a list of ip addresses and networks, and I used to have a yaml file with the network laid out. But a lot of it was out of date and any time I made a significant change (like adding vlans or tagged interfaces), the entire format would need to change.  I've also been planning to redo how my network is laid out and realized I would have to document everything before I could really work on the new scenario. But overall, these projects have been at the very bottom of the list, kind of kicked under the rug.

This weekend, I got a big itch to go back and figure out how everything was laid out physically, and document it. I decided I just wanted to grab a tool, even if it wasn't the ideal do everything tool, and start recording things. So Saturday night and early Sunday morning, I started re-researching network documentation tools. I was pretty dissapointed by what I found, at least in the open source world.

Let me explain my home network setup. I have the house, my batcave/office, and a shed. These all are networked together, and each building has a managed switch in it. I have cable internet coming into the house. From the comcast router, a wire goes into a managed switch on a "public" vlan. My gateway router sits in the batcave is also plugged into a managed switch on the same vlan. I have 4U wall-mount racks holding patch panels (and/or switches) a 40U rack in my batcave holding the gateway router, a switch, and a couple servers. I also have some unifi access points spread around the property.

To start off, I came up with a list of what I wanted to do:

* Enter in all of my physical devices
  * servers
  * racks
  * patch panels
  * switches
  * access points
  * routers
* Record the connection between each device.
* Note the native and tagged vlans on each switch port
* Possibly record server ip addresses, virtual machines, and their vlans (native or tagged)
* Be able to fetch my data by api, and ideally programatically enter
* Be able to view a rack elevation or cable path

What I did not want to do:

* Use a homemade file like a spreadsheet or yaml file (even though people have done wonders with making elevation spreadsheets)
* Write an application to do this
* Lock my data into an obscure format

## Research

**Spoiler Alert** As you can guess from the title, I ultimately installed openDCIM and used that. It doesn't meet all of my needs, but I'll explain my reasoning below.

After searching, I came up with there being several categories that touch upon these areas. They are 1) asset/inventory management, 2) network scanning and monitoring.

**Network Scanning and Monitoring:** Applications in this category, such as [opennms](http://www.opennms.org/) and [solarwinds npm](http://www.solarwinds.com/network-performance-monitor) work by interrogating your network and building a live map of everything connected. In reality, this is what most people, including myself should look at. The reason network documentation gets out of date is because someone is manually entering it. By providing a live report of the network (and showing historical changes in an audit log), you will always have the most accurate information. Looking at software in this category had me re-evaluate what I wanted to do. I was looking heavily at openNMS (and I've come across this product in the past). Here, the NMS stands for Network **Management** Solution, though they have a lot of focus on monitoring, so you might
assume the "M" stands for "Monitoring". openNMS looks excellent for what it does, and I will probably use openNMS down the road for my logical network documentation. But for what I wanted to do, openNMS and applications in this category are not designed to physically lay out hardware. There was nothing in openNMS about rack elevations or cable connectiosn that I could find. I even found people in forums looking for a way to integrate openNMS with Racktables (mentioned below).

Like I said, looking at applications in this category made me want to further separate my goals to target the physical layout. Some software can do live polling of switches to see what MAC addresses are connected to a port, and things like lldp will show friendly neighbor names. But it can't tell that there is a patch panel or unmanaged switch in between. The only way to get this information is visual inspection. I need an application that can do that.

**Asset and Inventory Management:** These also overlap with Config Management Databases. In fact, a number of IT Asset Managers also call themselves CMDBS, or vice versa. IT Asset Management is a pretty wide area. However, only a couple support concepts like rack elevations and
cable path management. The two I looked at the most were [RackTables](http://www.racktables.org/) and [opendcim](http://www.opendcim.org). A third one I looked at was [Ralph](http://ralph.allegro.tech/).

**Ralph:** Let me just talk about Ralph for a moment. I looked at Ralph two years ago for a larger project at work, and it was disqualified for a number of reasons, specific to that project. That experience gave me a negative view of Ralph, and I didn't give it too much looking over this time around. That may have been unwise. I took a second look while writing this up. If you look at their [documentation](http://ralph-ng.readthedocs.io/en/latest/), they seem to have all the features I'm looking for here, along with an API. It's based on python/django, and their [github](https://github.com/allegro/ralph) is pretty active. I think I owe it to Ralph to install and review the software again. A lot seems to have changed over the last two years.  **UPDATE:** I went and played with Ralph's demo. Very slick addings datacenters, server rooms, racks, and devices. If I go to add a device, I can create a new device template and manufacturer on the fly. However, it has no support for cable management. There is an ongoing [issue opened for this](https://github.com/allegro/ralph/issues/2225). So even though I didn't give Ralph a fair shake, it's out of the running for right now because it can't do links between interfaces.

**Racktables:** I've used racktables off an on over the years. Quite frankly, it's just not nice software to work with. Data entry is difficult, it has no native api support (though some people have worked at bolting some on), and in my mind, it's a one-way system. You put the data in and that's about it, you can only visually access the data afterwards. On the plus side, it does have some IPAM and VLAN management features, so for those looking to do more than physical layout, Racktables has quite an advantage.

**openDCIM:** Finally, we come to openDCIM. Like Racktables, it's php/mysql based. It has a rather nice interface for creating datacenters, cabinets, and devices. It understands about chassis and blade setups. It has a baked in read-only API. These days, my philosophy on web apps is that they should build an api, and then a frontend that uses that api. But this app pre-dates the popularity of APIs, and they have been adding it on afterwards. I would have been turned off by the lack of writeable API, but their existing html forms are basic enough that you could easily manipulate them with curl or python. If I really needed to update the data programatically, I am sure I could do so. But being able to run a curl command and get data back in json, means that I can easily integrate this with other tools down the road. Ultimately, I decided I didn't want to waste this motivation trying to seek out other tools and went with openDCIM. My goal for Sunday was to record what I could about the system.

## Actual Usage

### installation/vagrant

Installation was pretty straightforward. I did it inside a vagrant vm on my laptop. It was basically install Apache, PHP, Mysql and go to town. I
uploaded my [vagrant config to github](https://github.com/ytjohn/vagrant-opendcim), so you can clone that and start your own instance right away.
After installation, opendcim provides a web-based pre-fight check and walks you through creating datacenters and cabinets. This is pretty straightforward - you just give your datacenters a name, and your cabinets a name and u-height. Once done, you have to manually delete install.php
from the opendcim directory.

For me, I called my batcave one datacenter, and my house another. Then for each room, I made that a cabinet. In my basement, I have a 4U network rack on the wall, and a rack shelf also screwed into the wall (holding my synology and my comcast router). I called my shelf a 10U rack. Then for each room that has a wall jack and any equipment (like an access point) I wanted to track, I created an imaginary rack. I'll talk more about that in a bit. Now, at work, we use rack names like R01 or NP01 or A54. At home, I used highly technical names like "basementwallrack", "tvstand", and "batcaverack". Pick a naming scheme that works for you.

### enter manufacturers and templates

One of the first things I came across was that I can't just add a device to a rack. Devices are based on device templates, and templates are tied
to a manufacturer. This wasn't really a surprise, because almost any sort of asset tracker I've used works the same way. This means that I had to go
into my Template Management, and add manufacturers. Then I went into Template Management and start editing templates. This was a fun excercise going
into my emails, amazon orders, or just logging into a device to get a model number. In a template, you can define things like power consumption,
weight, network ports, and u-height. These are templates, so you won't be putting serial numbers in. I added templates for my managed switches,
my ubiquiti UAP and UAP-AC-PRO, 24v poe injectors, my chromebox, generic desktop computer, patch panels, and anything else I could think of.  There
was a neat looking feature where you could import templates and submit templates back, but none of the existing ones had my equipment. If you have
images of the front and back of a device, you can include those to make your rack elevations look more accurate.

One quick gothca was that when I started adding devices with network ports, I found I had to go into configuration->cabling types and add cable media
types like 1000BaseTX and 1000BaseFX. For fun, I added 802.11bgn along with 802.11ac. I also added a 1000BaseTX-POE24V medie type, because I have some
runs that are carrying 24V POE.

One useful to do when making templates is to go down to the Ports and rename them to things like "eth0". For my access ports, I made "LAN" and "WLAN" ports. For the POE injector template, I put "LAN" and "POE" as port names. You can always rename ports when you create a specific device to put in a rack, but the better your template, the less work later on.

Also, the device type is important. Most of the types (servers, storage arrays, and appliances) all work the same way. Physical Infrastructure does not have any ports. Patch Panels are unique in that each port ends up having a front and rear connection.

### adding devices and connecting ports

Finally, you can browse to a rack and start adding devies. When you add a device, you select from a template, add a label, and then select a u position. when you save that, you can then start connecting ports. I found it best to start with patch panels first. I have 24-port patch panels in
each wall-mounted network rack. I had to get creative for the wall jacks, and I made either 1-port or 2-port RJ45 keystone jacks (RJ4KJ1 and RJ45KJ2). When connecting patch panels (or wall jacks) make sure that you connect the rear of one patch panel to the rear of another. When editing a port, you can "connect" the front and rear side of the port at the same time. So you can connect the front of the patch panel to a switchport, and the back of your patch panel port to the back of a wall jack, and then hit save.

I found that saving connections seemed straightforward, but was easy to make mistakes. After you have entered each row and clicked save on it, you need to hit "update" to save all of your changes. Also, if you don't save your rows before hitting update, your changes will be lost. I also found that once I linked two ports, I could no longer change the name of the ports. Once you make connections, you can see the entire path in an image form, or in a text description like this; `SW02BATCAVE[Port4]BATCAVE-PATCH24[Port4]BC2[BC2-2]FRESHDESK[eth0]`.

As for IP addresses and multiple interfaces, this was sadly lacking. I could enter a management address for a device. On the ports, there is a notes column. I could add an ip address or a vlan bit there, but it's simply a free-form field.

**snmp** When adding switches, if I added an ip address, I could query the switch with snmp. On my tp-link switches, it was able to get basic system information over snmp, but it could not get a list of ports. If it had, I believe it can populate some port status information.


### imaginary racks and other oddities

As I mentioned above, I had to make imaginary racks in each room.  The imaginary racks were sort of a pain point for me. I get that this program was written with racks in mind. The concept of a freestanding device such as a celing mounted access point, a wall jack, a printer, or a desktop tower just really doesn't factor in. The idea is that you have racks, and only devices that are in a rack can be cabled.

This also impacted how I made wall jacks. A single port wall jack, I had to enter as a patch panel, 1U in height. If a device does not have a U-height, you can't add it to a rack. And if you don't add it to a rack, you can't cable it. So, in order to document the RJ45KJ1 and RJ45KJ2, I created two 10U racks in my living room. "TVSTAND" with a 1U RJ45KJ1" and "LRCOUCH" with a 1U "RJ45KJ2". For TVSTAND, I added my tp-link unmanaged switch, my UAP-AC-PRO, and (for fun), my Chromebox. The switch connects to the front of the RJ45KJ1. The Chromebox connects to the unmanaged switch.

My access point provided another hitch. This might be a bit obsessive, but I want to record when something is using POE. So I created a Ubiquiti
24VPOEINJECTOR appliance template, which I used to create a (1U) device to place in my imaginary rack. One port (LAN) connects to the switch, while the other (POE) connects to the access point.

For my living room, since the POE lives with the access point, this isn't really needed. But for the access point in the hallway, the poe injection takes place in the basement, and we have a 1000BaseTX running from the switch to the injector, then 1000BaseTX-POE24V running to the front of the patch panel, then from the rear of the patch panel, to the keystone jack in the wall, and finally up to the access point.  I have a similar setup in my batcave, with a POE injector powering an external access point, and another (48V!) powering an ip phone. While POE is supposed to be safe for non POE devices, I think it comes in handy to document which wall jack I can expect to find power at.

## Wrap Up

This about wraps up my experience. Over the course of a Sunday, I was able to get openDCIM up and running, and enter all the data that describes the physical layout of my network. I would love to be able to wire up freestanding devices in a data center, and I would like to assign ip addresses and vlans to individual interfaces. But for physical layout and inventory, it works really well. I suspect that another application like openNMS will have to track my logical network. If it can be configured to query my switches (snmp/lldp), then it would be a better live solution. Ralph might be a good system for handling this aspect as well, though that requires further investigation.

Once all the data was in, I was able to do curl commands and retrieve json. Since references to other devices were id numbers, a true fech and sync would need to make multiple calls, retrieving related records. For visualization, openDCIM has a reports feature, including a network map. This network map is generated using [graphviz dot language](http://www.graphviz.org/content/dot-language), and it can output that in png or svg. The default generated map is a bit difficult to trace lines from port to port. But I took the dot file, changed `splines` to `ortho` and it came out much nicer. I think there's room for improvement here, and I think with some tweaking, we can make a really nice printable network diagram to hang up next to each rack.

Another feature that might be nice would be printable asset labels, that have a QR pointing back to the opendcim instance. With the API, I could definitely see writing a script to pull and generate these.

I used mysqldump to backup my data and I can run this like an application, though I plan to put this on a VM. My next goal (in this category) is to create an ansible role to install this on one of my virtual machines and give it an always on life.
