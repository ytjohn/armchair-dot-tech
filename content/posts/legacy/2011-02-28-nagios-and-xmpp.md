---

title: Nagios and XMPP
author: ytjohn
date: 2011-02-28 17:58:10

layout: post

slug: nagios-and-xmpp

---
I found that someone has written a perl script geared towards sending
alerts from Nagios to XMPP usernames.

http://www.gridpp.ac.uk/wiki/Nagios_jabber_notification

I have downloaded this, but have not yet got it working as of yet, but
it does look promising.  It took me a while to update all the
dependencies for it, but if those were in place, the installation itself
is rather simple.  That is, the notification script works, but I haven't
actually configured nagios as of yet.

This script has a shortcoming in that it only accepts the username
portion of the JID and not the domain name -- and this means that
notifications can only be sent between users of the same domain.

To illustrate, user@example.net can not send a message to
user@example.com, but user1 and user2 on example.net can send to each
other.

<blockquote>
# this command will fail:<br />
ytjohn@monitor:/usr/local/bin$ ./notify_by_jabber.pl
yourtech@example.com testing  <br />
# while this one works<br />
ytjohn@monitor:/usr/local/bin$ ./notify_by_jabber.pl yourtech
testing</br></br></br>
</blockquote>

And in the perl script, you have to specify the login credentials and
server you're connecting to:

<blockquote>
## Configuration<br />
# my $username = 'system@example.com';   # does not work<br />
my $username = 'system';<br />
my $password = "password";<br />
my $resource = "nagios";<br />
## End of configuration  </br></br></br></br></br>

my $len = scalar @ARGV;<br />
if ($len ne 2) {<br />
   die "Usage...n $0 [jabberid] [message]n";<br />
}<br />
my @field=split(/,/,$ARGV[0]);<br />
#------------------------------------  </br></br></br></br></br>

# Google Talk &amp; Jabber parameters :  

my $hostname = 'talk.google.com';<br />
my $port = 5222;<br />
# componentname is the second half of your JID:<br />
my $componentname = 'example.com';<br />
my $connectiontype = 'tcpip';<br />
my $tls = 1;</br></br></br></br></br>
</blockquote>
