---

title: python console import tip
author: ytjohn
date: 2012-08-29 17:58:11

layout: post

slug: python-console-import-tip

---
Tags: python console, python, import, pysphere

This is a quick little python tip. When experimenting with python commands and modules, it's usually easiest to use the python console interactively, then create your programs later. The downside of this is that sometimes you have to do a bit of typing before you get to the specific command you want to try.

Imagine the following example:

<pre><code>johnh@puppet2:~$ python
Python 2.7.3 (default, Aug  1 2012, 05:14:39)
[GCC 4.6.3] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>&gt;&gt; from pysphere import *
>&gt;&gt; server = VIServer()
>&gt;&gt; server.connect("vc1.example.com", "username", "password")
>&gt;&gt; print server.get_server_type(), server.get_api_version()
VMware vCenter Server 5.0
</code></pre>

Here, I had to type in 3 lines, including my password in plaintext, to test out querying the server. I can't demonstrate this live, because then I reveal my password. Well last week, I made a [test1.py] file that reads a yaml configuration file and does the commands I just did. Here's the smart bit. I can import that file directly into the python console. Once it imports, it runs each python command and leaves me in the console, ready to query the system again. The only caveat is that my "server" variable is now part of the test1 module as "test1.server".

<pre><code>johnh@puppet2:~$ python
Python 2.7.3 (default, Aug  1 2012, 05:14:39)
[GCC 4.6.3] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>&gt;&gt; import test1
VMware vCenter Server 5.0
puppet1-centos6 10.100.0.206
>&gt;&gt; test1.server.is_connected()
True
>&gt;&gt; vmlist = test1.server.get_registered_vms()
>&gt;&gt; for v in vmlist:
...     print v
...
[iscsi-244gb-raid10] rhel-puppet1/rhel-puppet1.vmx  
[iscsi-244gb-raid10] puppet2-ubuntu 12.04LTS server/puppet2-ubuntu 12.04LTS server.vmx
[iscsi-488gb-raid10] puppet3-solaris11/puppet3-solaris11.vmx
</code></pre>
