---

title: Jabbering about Python
author: ytjohn
date: 2011-02-27 17:58:10

layout: post

slug: jabbering-about-python

---
I am getting more and more into the idea of using XMPP for sending
messages.  XMPP (also known as jabber) is a very open protocol.  Anyone
can throw up a server, and by publishing a few DNS records, your server
can interact with any other XMPP server out there (that is open).  For a
popular example, anyone can throw up their server with their domain and
send messages to someone using Google Chat.<br />
<br />
I have a customer that has their domain email hosted by Google apps for
domains, which also means that each person has Google Talk, which is
based off of XMPP.  For a monitoring scenario, I decided I'd rather have
alerts go to an installed Google Talk client instead of to their email. 
This would make alerts more noticeable, but less intrusive to their
Inbox.  Google would also tie all their messages together (chat and
email) for searching later. So my idea was to have monitoring server
send the email alert to a local user, and have it piped through a
program that would send the XMPP alert.  In the future, perhaps an XMPP
bot could also accept commands to control the monitoring system.<br />
<br />
I wanted to create a program called "jabblast.py" which would accept
either standard input, or a command line argument, and send a message to
a pre-defined list of recipients.  </br></br></br></br>

<blockquote>
echo "sent from stdin" | jabblast.py<br />
  jabblast.py -m "sent from a command line argument"</br>
</blockquote>

Looking at the python and XMPP options out there, I came across three
that looked promising. 

<blockquote>
# xmpppy<br />
  # sleekxmpp<br />
  # twisted (the "words" extension)</br></br>
</blockquote>

I was really expecting to find exactly what I wanted to do already
written.  However, it seems anyone writing an XMPP script is creating a
bot that runs continuously and would need modified for a one-time
event.<br />
<br />
Sleekxmpp was actually the easiest to understand and work with, however
the project could be in danger of abandonment.   Twisted looks the most
advanced, and I am drawn to it because it seems to offer the most
versatility.  In my mind, I suspect that if I learn twisted for this, I
can just keep using it for all the other stuff I want to do.  However,
it might be "too much" for this simple of an operation.  xmpppy is the
middle ground.  The project seems relatively active and popular.    </br></br>

The problem with all three projects is a severe lack of documentation --
for twisted, this lack is primarily on the xmpp side.  One thing very
well hidden is how to specify the server name you want to connect to. 
Ideally, this is provided through a DNS record lookup, but that method
eliminates experimentation with local network ip addresses.   Sleek
never even approaches the concept that you might want your XMPP program
to end.   It has a call for disconnecting, but when I call that, sleek
immediately tries to reconnect.

I'm going to go ahead and create my program using all three methods and
decide later which I like best.

For these examples, I have hardcoded the login credentials, as well as
the "to" into the source code itself.  In the real world, I should be
pulling that from a configuration file, either XML or YAML (and
potentially from a remote web page).  Secondly, I only list one "to".  I
believe that going from sending a single message to sending multiple
messages should be a matter of a for loop.  Some libraries might even
offer a feature to send to multiple recipients at once.

For my first example, I have created a script in sleekxmpp.  I wrote
this by looking at the EchoBot example found <a href="https://github.com/remko/xmpp-tdg/tree/master/code/EchoBot">here</a>.  The script will
send a message and then hang for a while after disconnecting before it
finally exits.  I don't know why.  If you enable logging, an error is
thrown after you pass in the disconnect command.  But otherwise, a
relatively simple bit of code, easy to expand upon.

<blockquote>
#!/usr/bin/env python  

import sys<br />
import logging<br />
import sleekxmpp  </br></br>

# Uncomment the following line to turn on debugging<br />
#logging.basicConfig(level=logging.DEBUG, format="%(levelname)-8s
%(message)s")  </br>

def main() : <br />
  # configure jabber id credentials and to<br />
  to = 'recipient@example.net'<br />
  myjid = 'sender@example.net/blastbot'<br />
  mysecret = 'password'  </br></br></br></br>

args = sys.argv[1:]<br />
  if not args:<br />
    message = sys.stdin.read()<br />
  else:<br />
    message = sys.argv[1]  </br></br></br></br>

# print to, message<br />
  # sys.exit()  </br>

bot = BlastBot(myjid, mysecret, to, message)<br />
  bot.run()   </br>

class BlastBot :   

def __init__(self, jid, password, to, message) : <br />
    self.message = message<br />
    self.to = to<br />
    self.xmpp = sleekxmpp.ClientXMPP(jid, password) <br />
    self.xmpp.add_event_handler("session_start",
self.handleXMPPConnected)<br />
    self.xmpp.add_event_handler("disconnected",
self.handleXMPPDisconnected)  </br></br></br></br></br>

def run(self) :<br />
    self.xmpp.connect() <br />
    self.xmpp.process(threaded=False)   </br></br>

def handleXMPPDisconnected(self, event) :<br />
    sys.exit()  </br>

def handleXMPPConnected(self, event) :<br />
    self.xmpp.sendMessage(self.to, self.message)<br />
    self.xmpp.disconnect(self)  </br></br>

if __name__ == "__main__" :<br />
  main()</br>
</blockquote>
