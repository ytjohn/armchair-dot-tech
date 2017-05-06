---
ID: 89
post_title: Sending messages using xmpppy
author: ytjohn
post_date: 2011-04-05 15:00:00
post_excerpt: ""
layout: post
permalink: >
  http://devblog.yourtech.us/2011/04/05/sending-messages-using-xmpppy/
published: true
---
In continuing working with XMPP and Python, I managed to get xmpppy
working.

This proved to be particularly tricky because xmpppy uses some outdated
modules like md5 and sha.  It also relies on some dns functions which no
longer seem to work.

<blockquote>
I was able to use the sample script "<a href="http://xmpppy.sourceforge.net/examples/xsend.py">xsend.py</a>" that the project
provides.  The big thing is that I had to specify talk.google.com and
the port number directly in the connect command.
</blockquote>

If I did not specify the servername, I would get the error:<br />
 An error occurred while looking up _xmpp-client._tcp.example.com  </br>

NOTE: DNS is configured correctly for the domain I was practicing with,
but the xmpppy code was not able to resolve it.

To run the script, simply execute;

<blockquote>
./xblast.py destination@example.com  text to send
</blockquote>

The Script:

<blockquote>
#!/usr/bin/python<br />
# \$Id: xsend.py,v 1.8 2006/10/06 12:30:42 normanr Exp \$<br />
import sys,os,xmpp,time  </br></br>

if len(sys.argv) \&lt; 2:<br />
    print "Syntax: xsend JID text"<br />
    sys.exit(0)  </br></br>

tojid=sys.argv[1]<br />
text=' '.join(sys.argv[2:])  </br>

jidparams={}  

jidparams['jid']='system@example.com'<br />
jidparams['password'] = '123456'  </br>

jid=xmpp.protocol.JID(jidparams['jid'])<br />
cl=xmpp.Client(jid.getDomain(),debug=True)  </br>

con=cl.connect(('talk.google.com',5222),  use_srv=False)  

if not con:<br />
    print 'could not connect!'<br />
    sys.exit()<br />
print 'connected with',con<br />
auth=cl.auth(jid.getNode(),jidparams['password'],resource=jid.getResource())<br />
if not auth:<br />
    print 'could not authenticate!'<br />
    sys.exit()<br />
print 'authenticated using',auth  </br></br></br></br></br></br></br></br>

# cl.SendInitPresence(requestRoster=0)   # you may need to uncomment
this for old server<br />
id=cl.send(xmpp.protocol.Message(tojid,text))<br />
print 'sent message with id',id  </br></br>

time.sleep(1)   # some older servers will not send the message if you
disconnect immediately after sending  

cl.disconnect()
</blockquote>

<hr/>