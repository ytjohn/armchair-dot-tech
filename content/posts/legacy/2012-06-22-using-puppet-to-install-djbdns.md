---

title: Using puppet to install djbdns
author: ytjohn
date: 2012-06-22 17:58:11

layout: post

slug: using-puppet-to-install-djbdns

---
This is a basic walkthrough of getting a slightly complex "step by step
to install" program like djbdns to install under puppet (in this case,
under Ubuntu 12.04). It shows building the manifest, testing it, and
some possible gotchas.

I am generally following the guide put together by Higher Logic[1], with
a few changes of my own.

Step 1: Installation<br />
I use the dbndns fork of djbdns, which has a few patches installed that
djbdns lacks. In fact, the djbdns package in Debian/Ubuntu is a virtual
package that really install dbndns. To install it normally, you would
type "sudo apt-get install dbndns". This would also install daemontools
and daemontools-run. However, we'll also need make and ucspi-tcp.  </br>

We're going to do this the puppet way. I'm assuming my puppet
configuration in in /etc/puppet, node manifests are in
/etc/puppet/nodes, and modules are in /etc/puppet/modules.

a. Create the dbndns module with a package definition to install

sudo mkdir -p /etc/puppet/modules/dbndns/manifests<br />
    sudo vi /etc/puppet/modules/dbndns/manifests/init.pp<br />
<br />
        class dbndns {<br />
            package {<br />
                    dbndns:<br />
                    ensure =&gt; present;  </br></br></br></br></br></br>

ucspi-tcp:<br />
                    ensure =&gt; present;  </br>

make:<br />
                    ensure =&gt; present;<br />
            }  </br></br>

}

b. Create a file for your node (ie: puppet2.example.net)

sudo vi /etc/puppet/nodes/puppet2.example.net.pp<br />
<br />
        node    'puppet2.lab.example.net' {<br />
            include dbndns<br />
        }<br />
</br></br></br></br></br>

c. Test<br />
Ok, to test on your puppet client, run "sudo puppet agent --test"  </br>

johnh@puppet2:~# sudo puppet agent --test<br />
    info: Retrieving plugin<br />
    info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb<br />
    info: Caching catalog for puppet2.lab.example.net<br />
    info: Applying configuration version '1340213237'<br />
    notice: /Stage[main]/Dbndns/Package[dbndns]/ensure: created<br />
    notice: Finished catalog run in 3.39 seconds  </br></br></br></br></br></br></br></br>

Here we can see our dbndns package installed. But is it running? Well,
djbdns uses daemontools, which runs svscan, and some searching online
shows that in Ubuntu 12.04/Precise, this is now an upstart job. svscan
is not running. So let's make it run. Add the following to your init.pp
(within the module definition):

<h1>define the service to restart<br /></h1>

        service { "svscan":<br />
                ensure  =&gt; "running",<br />
                provider =&gt; "upstart",<br />
                require =&gt; Package["dbndns"],<br />
        }  </br></br></br></br></br>

Now back on puppet2, let's test it.

johnh@puppet2:~# sudo puppet agent --test<br />
    info: Retrieving plugin<br />
    info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb<br />
    info: Caching catalog for puppet2.lab.example.net<br />
    info: Applying configuration version '1340213237'<br />
    notice: /Stage[main]/Dbndns/Service[svscan]/ensure: ensure changed
'stopped' to 'running'<br />
    notice: Finished catalog run in 0.47 seconds  </br></br></br></br></br></br></br></br>

We now told puppet to ensure that svscan is running. The "provider"
option tells it to use upstart instead of /etc/init.d/ scripts or the
service command. Also, we make sure that it doesn't attempt to start
svscan unless dbndns is already installed.

Now we have daemontools running, but we haven't got it start our tinydns
service yet. To do that, we need to create some users and configure the
service.<br />
<br />
Step 2: Create users  </br></br>

Going back to our guide, our next step is to create users. We can do
that in puppet as well.<br />
    # Users for the chroot jail<br />
    adduser --no-create-home --disabled-login --shell /bin/false dnslog<br />
    adduser --no-create-home --disabled-login --shell /bin/false
tinydns<br />
    adduser --no-create-home --disabled-login --shell /bin/false
dnscache  </br></br></br></br>

So in our init.pp module file, we need to define our users:

user { "dnslog":<br />
            shell =&gt; "/bin/false",<br />
            managehome =&gt; "no",<br />
            ensure =&gt; "present",<br />
        }<br />
<br />
    user { "tinydns":<br />
            shell =&gt; "/bin/false",<br />
            managehome =&gt; "no",<br />
            ensure =&gt; "present",<br />
        }<br />
<br />
    user { "dnscache":<br />
            shell =&gt; "/bin/false",<br />
            managehome =&gt; "no",<br />
            ensure =&gt; "present",<br />
        }  </br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br>

Back on puppet2, we can give that a test.

johnh@puppet2:~$ sudo puppet agent --test<br />
    info: Retrieving plugin<br />
    info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb<br />
    info: Caching catalog for puppet2.lab.example.net<br />
    info: Applying configuration version '1340215757'<br />
    notice: /Stage[main]/Dbndns/User[dnscache]/ensure: created<br />
    notice: /Stage[main]/Dbndns/User[tinydns]/ensure: created<br />
    notice: /Stage[main]/Dbndns/User[dnslog]/ensure: created<br />
    notice: Finished catalog run in 0.86 seconds<br />
    johnh@puppet2:~$ cat /etc/passwd | grep dns<br />
    dnscache:x:1001:1001::/home/dnscache:/bin/false<br />
    tinydns:x:1002:1002::/home/tinydns:/bin/false<br />
    dnslog:x:1003:1003::/home/dnslog:/bin/false  </br></br></br></br></br></br></br></br></br></br></br></br></br></br>

So far, so good. Now we have to do the configuration, which will require
executing some commands.

Step 3 - Configuration<br />
Our next step are the following commands:  </br>

<h1>Config<br /></h1>

    tinydns-conf tinydns dnslog /etc/tinydns/ 1.2.3.4<br />
    dnscache-conf dnscache dnslog /etc/dnscache 127.0.0.1<br />
    cd /etc/dnscache; touch /etc/dnscache/root/ip/127.0.0<br />
    mkdir /etc/service ; cd /etc/service ; ln -sf /etc/tinydns/ ; ln -sf
/etc/dnscache<br />
<br />
The first two commands create our service directories. Authoratative
tinydns is set to listen on 1.2.3.4 and dnscache is set to listen on
127.0.0.1. The 3rd command creates a file that restricts dnscache to
only respond to requests from IPs starting with 127.0.0. This is isn't
necessary, but the challenge is interesting.  </br></br></br></br></br></br>

What we want to do first is see if /etc/tinydns and /etc/dnscache exist
and if not, run the -conf program. We also need to know the IP address.
Fortunately, puppet provides this as a variable "$ipaddress". Try
running the "facter" command.

Puppet has a property call creates that is ideal. If the directory
specified by creates does not exist, it will perform the associated
commands. Here are our new lines:

exec { "configure-tinydns":<br />
            command =&gt; "/usr/bin/tinydns-conf tinydns dnslog
/etc/tinydns $ipaddress",<br />
            creates =&gt; "/etc/tinydns",<br />
            require =&gt; Package['dbndns'],<br />
    }  </br></br></br></br>

exec { "configure-dnscache":<br />
            command =&gt; "/usr/bin/dnscache-conf dnscache dnslog
/etc/dnscache 127.0.0.1",<br />
            creates =&gt; "/etc/dnscache",<br />
            require =&gt; Package['dbndns'],<br />
    }  </br></br></br></br>

Thos will configure tinydns and dnscache, and then we can restrict
dnscache

file { "/etc/dnscache/root/ip/127.0.0":<br />
            ensure =&gt; "present",<br />
            owner =&gt; "dnscache",<br />
            require =&gt; Exec["configure-dnscache"],<br />
    }  </br></br></br></br>

Then, we need to create the /etc/service directory and bring tinydns and
dnscache under svscan's control.<br />
<br />
    file { "/etc/service":<br />
            ensure =&gt; "directory",<br />
            require =&gt; Package["dbndns"],<br />
    }  </br></br></br></br></br>

file { "/etc/service/tinydns":<br />
            ensure =&gt; "link",<br />
            target =&gt; "/etc/tinydns",<br />
            require =&gt; [ File['/etc/service'],
Exec["configure-tinydns"], ],<br />
    }  </br></br></br></br>

file { "/etc/service/dnscache":<br />
            ensure =&gt; "link",<br />
            target =&gt; "/etc/dnscache",<br />
            require =&gt; [  File['/etc/service'],
Exec["configure-dnscache"]  ],<br />
    }  </br></br></br></br>

And our tests:

johnh@puppet2:~$ sudo puppet agent --test<br />
    info: Retrieving plugin<br />
    info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb<br />
    info: Caching catalog for puppet2.lab.example.net<br />
    info: Applying configuration version '1340218775'<br />
    notice: /Stage[main]/Dbndns/Exec[configure-dnscache]/returns:
executed successfull<br />
    notice:
/Stage[main]/Dbndns/File[/etc/dnscache/root/ip/127.0.0]/ensure: created<br />
    notice: /Stage[main]/Dbndns/File[/etc/service/dnscache]/ensure:
created<br />
    notice: /Stage[main]/Dbndns/Exec[configure-tinydns]/returns:
executed successfully<br />
    notice: /Stage[main]/Dbndns/File[/etc/service/tinydns]/ensure:
created<br />
    notice: Finished catalog run in 0.59 seconds<br />
    johnh@puppet2:~$ ls /etc/service/tinydns/root/<br />
    add-alias  add-alias6  add-childns  add-host  add-host6  add-mx 
add-ns  data  Makefile<br />
    johnh@puppet2:~$ ps ax | grep supervise<br />
     7932 ?        S      0:00 supervise dnscache<br />
     7933 ?        S      0:00 supervise log<br />
     7934 ?        S      0:00 supervise tinydns<br />
     7935 ?        S      0:00 supervise log  </br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br>

Doing a dig www.example.net @localhost returns 192.0.43.10, so dnscache
works.

Now, let's check tinydns. No domains are configured yet, so let's put
example.com in there. Edit /etc/tinydns/root/data and put these lines in
it, substituting 10.100.0.178 for your own "public" IP address.

<blockquote>
&amp;example.com::ns0.example.com.:3600 
Zexample.com:ns0.example.com.:hostmaster.example.com.:1188079131:16384:2048:1048576:2560:2560<br />
 +ns0.example.com:10.100.0.178:3600</br>
</blockquote>

Then "make" the data.cdb file:

cd /etc/tinydns/root ; sudo make<br />
<br />
Now test:  </br></br>

johnh@puppet2:/etc/tinydns/root$ dig ns0.example.com @10.100.0.178

; &lt;&lt;&gt;&gt; DiG 9.8.1-P1 &lt;&lt;&gt;&gt; ns0.example.com @10.100.0.178<br />
    ;; global options: +cmd<br />
    ;; Got answer:<br />
    ;; -&gt;&gt;HEADER&lt;&lt;- opcode: QUERY, status: NOERROR, id: 25433<br />
    ;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL:
0<br />
    ;; WARNING: recursion requested but not available  </br></br></br></br></br>

;; QUESTION SECTION:<br />
    ;ns0.example.com.               IN      A  </br>

;; ANSWER SECTION:<br />
    ns0.example.com.        3600    IN      A       10.100.0.178  </br>

;; AUTHORITY SECTION:<br />
    example.com.            3600    IN      NS      ns0.example.com.  </br>

Ok, for a final test, let's remove everything and run it again.

sudo service svscan stop<br />
    sudo apt-get purge daemontools daemontools-run ucspi-tcp dbndns<br />
    sudo rm -rf /etc/service /etc/tinydns /etc/dnscache<br />
    sudo userdel tinydns <br />
    sudo userdel dnslog <br />
    sudo userdel dnscache<br />
<br />
Let's do this:  </br></br></br></br></br></br></br>

johnh@puppet2:~$ sudo puppet agent --test<br />
    info: Retrieving plugin<br />
    info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb<br />
    info: Caching catalog for puppet2.lab.example.net<br />
    info: Applying configuration version '1340220032'<br />
    notice: /Stage[main]/Dbndns/Service[svscan]/ensure: ensure changed
'stopped' to 'running'<br />
    err: /Stage[main]/Dbndns/Exec[configure-dnscache]/returns: change
from notrun to 0 failed: /usr/bin/dnscache-conf dnscache dnslog
/etc/dnscache 127.0.0.1 returned 111 instead of one of [0] at
/etc/puppet/modules/dbndns/manifests/init.pp:47<br />
    notice: /Stage[main]/Dbndns/User[dnscache]/ensure: created<br />
    notice: /Stage[main]/Dbndns/User[tinydns]/ensure: created<br />
    notice: /Stage[main]/Dbndns/File[/etc/service]/ensure: created<br />
    notice: /Stage[main]/Dbndns/File[/etc/service/dnscache]: Dependency
Exec[configure-dnscache] has failures: true<br />
    warning: /Stage[main]/Dbndns/File[/etc/service/dnscache]: Skipping
because of failed dependencies<br />
    notice: /Stage[main]/Dbndns/User[dnslog]/ensure: created<br />
    notice: /Stage[main]/Dbndns/File[/etc/dnscache/root/ip/127.0.0]:
Dependency Exec[configure-dnscache] has failures: true<br />
    warning: /Stage[main]/Dbndns/File[/etc/dnscache/root/ip/127.0.0]:
Skipping because of failed dependencies<br />
    notice: /Stage[main]/Dbndns/Exec[configure-tinydns]/returns:
executed successfully<br />
    notice: /Stage[main]/Dbndns/File[/etc/service/tinydns]/ensure:
created<br />
    notice: Finished catalog run in 0.98 seconds  </br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br>

Looks like we had something fail. Oops! configure-dnscache failed. We
see that the user dnscache and tinydns were created after. So we need to
make sure that the users are created before we can configure the
service. This needs to happen to tinydns as well as dnscache. Good thing
we did this test so it doesn't bite us in the future. Let's adjust our
init.pp

exec { "configure-tinydns":<br />
                command =&gt; "/usr/bin/tinydns-conf tinydns dnslog
/etc/tinydns $ipaddress",<br />
                creates =&gt; "/etc/tinydns",<br />
                require =&gt; [ Package['dbndns'], User['dnscache'],
User['dnslog'] ],<br />
        }  </br></br></br></br>

exec { "configure-dnscache":<br />
                command =&gt; "/usr/bin/dnscache-conf dnscache dnslog
/etc/dnscache 127.0.0.1",<br />
                creates =&gt; "/etc/dnscache",<br />
                require =&gt; [ Package['dbndns'],  User['dnscache'],
User['dnslog'] ],<br />
        }  </br></br></br></br>

Also, let's go ahead and run our commands above to get rid of everything
again.

johnh@puppet2:~$ sudo puppet agent --test<br />
    info: Retrieving plugin<br />
    info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb<br />
    info: Caching catalog for puppet2.lab.example.net<br />
    info: Applying configuration version '1340220641'<br />
    notice: /Stage[main]/Dbndns/Service[svscan]/ensure: ensure changed
'stopped' to 'running'<br />
    notice: /Stage[main]/Dbndns/User[dnscache]/ensure: created<br />
    notice: /Stage[main]/Dbndns/User[tinydns]/ensure: created<br />
    notice: /Stage[main]/Dbndns/File[/etc/service]/ensure: created<br />
    notice: /Stage[main]/Dbndns/User[dnslog]/ensure: created<br />
    notice: /Stage[main]/Dbndns/Exec[configure-dnscache]/returns:
executed successfully<br />
    notice: /Stage[main]/Dbndns/File[/etc/service/dnscache]/ensure:
created<br />
    notice:
/Stage[main]/Dbndns/File[/etc/dnscache/root/ip/127.0.0]/ensure: created<br />
    notice: /Stage[main]/Dbndns/Exec[configure-tinydns]/returns:
executed successfully<br />
    notice: /Stage[main]/Dbndns/File[/etc/service/tinydns]/ensure:
created<br />
    notice: Finished catalog run in 1.05 seconds  </br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br>

Everything looks good, but when we run "ps ax | grep svscan" we don't
see svscan running. So we check /var/log/syslog and see this

Jun 20 19:31:35 puppet2 kernel: [ 9646.348251] init: svscan main
process ended, respawning<br />
    Jun 20 19:31:35 puppet2 kernel: [ 9646.359074] init: svscan
respawning too fast, stopped<br />
<br />
If we start it by hand, it works, so what happened? /etc/service didn't
exist yet.  </br></br></br>

johnh@puppet2:~$ sudo service svscan start<br />
    svscan start/running, process 9726<br />
    johnh@puppet2:~$ ps ax | grep supervise<br />
     9730 ?        S      0:00 supervise dnscache<br />
     9731 ?        S      0:00 supervise log<br />
     9732 ?        S      0:00 supervise tinydns<br />
     9733 ?        S      0:00 supervise log  </br></br></br></br></br></br>

Let's fix that.

<h1>define the service to restart<br /></h1>

        service { "svscan":<br />
                ensure  =&gt; "running",<br />
                provider =&gt; "upstart",<br />
                require =&gt; [ Package["dbndns"], File["/etc/service"] ]<br />
        }  </br></br></br></br></br>

Now, let's give it a go:

johnh@puppet2:~$ sudo puppet agent --test<br />
    info: Retrieving plugin<br />
    info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb<br />
    info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb<br />
    info: Caching catalog for puppet2.lab.example.net<br />
    info: Applying configuration version '1340220885'<br />
    notice: /Stage[main]/Dbndns/User[dnscache]/ensure: created<br />
    notice: /Stage[main]/Dbndns/User[tinydns]/ensure: created<br />
    notice: /Stage[main]/Dbndns/File[/etc/service]/ensure: created<br />
    notice: /Stage[main]/Dbndns/Service[svscan]/ensure: ensure changed
'stopped' to 'running'<br />
    notice: /Stage[main]/Dbndns/User[dnslog]/ensure: created<br />
    notice: /Stage[main]/Dbndns/Exec[configure-dnscache]/returns:
executed successfully<br />
    notice: /Stage[main]/Dbndns/File[/etc/service/dnscache]/ensure:
created<br />
    notice:
/Stage[main]/Dbndns/File[/etc/dnscache/root/ip/127.0.0]/ensure: created<br />
    notice: /Stage[main]/Dbndns/Exec[configure-tinydns]/returns:
executed successfully<br />
    notice: /Stage[main]/Dbndns/File[/etc/service/tinydns]/ensure:
created<br />
    notice: Finished catalog run in 1.24 seconds<br />
    johnh@puppet2:~$ ps ax | grep svscan<br />
    10613 ?        Ss     0:00 /bin/sh /usr/bin/svscanboot<br />
    10615 ?        S      0:00 svscan /etc/service<br />
    10639 pts/0    S+     0:00 grep --color=auto svscan<br />
    johnh@puppet2:~$ ps ax | grep supervise<br />
    10630 ?        S      0:00 supervise dnscache<br />
    10631 ?        S      0:00 supervise log<br />
    10632 ?        S      0:00 supervise tinydns<br />
    10633 ?        S      0:00 supervise log<br />
    10641 pts/0    S+     0:00 grep --color=auto supervise  </br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br>

Excellent! We now have a working puppet class that will install puppet,
configure it, and get it up and running. At this point, we don't have
any records being served by tinydns, but it wouldn't be hard to push a
file to /etc/tinydns/root/data and execute a command to perform the
make. In my case, I will be using VegaDNS's update-data.sh[2] to pull
the data remotely.

Here is our completed modules/dbndns/init.pp:

</p>

<hr />

class dbndns {

package {<br />
                dbndns:<br />
                ensure =&gt; present;  </br></br>

ucspi-tcp:<br />
                ensure =&gt; present;  </br>

make:<br />
                ensure =&gt; present;<br />
        }  </br></br>

<h1>define the service to restart<br /></h1>

        service { "svscan":<br />
                ensure  =&gt; "running",<br />
                provider =&gt; "upstart",<br />
                require =&gt; [ Package["dbndns"], File["/etc/service"] ]<br />
        }  </br></br></br></br></br>

user { "dnslog":<br />
                        shell =&gt; "/bin/false",<br />
                        managehome =&gt; false,<br />
                        ensure =&gt; "present",<br />
                }  </br></br></br></br>

user { "tinydns":<br />
                        shell =&gt; "/bin/false",<br />
                        managehome =&gt; false,<br />
                        ensure =&gt; "present",<br />
                }  </br></br></br></br>

user { "dnscache":<br />
                        shell =&gt; "/bin/false",<br />
                        managehome =&gt; false,<br />
                        ensure =&gt; "present",<br />
                }  </br></br></br></br>

exec { "configure-tinydns":<br />
                command =&gt; "/usr/bin/tinydns-conf tinydns dnslog
/etc/tinydns $ipaddress",<br />
                creates =&gt; "/etc/tinydns",<br />
                require =&gt; [ Package['dbndns'], User['dnscache'],
User['dnslog'] ],<br />
        }  </br></br></br></br>

exec { "configure-dnscache":<br />
                command =&gt; "/usr/bin/dnscache-conf dnscache dnslog
/etc/dnscache 127.0.0.1",<br />
                creates =&gt; "/etc/dnscache",<br />
                require =&gt; [ Package['dbndns'],  User['dnscache'],
User['dnslog'] ],<br />
        }  </br></br></br></br>

file { "/etc/dnscache/root/ip/127.0.0":<br />
                ensure =&gt; "present",<br />
                owner =&gt; "dnscache",<br />
                require =&gt; Exec["configure-dnscache"],<br />
        }  </br></br></br></br>

file { "/etc/service":<br />
                ensure =&gt; "directory",<br />
                require =&gt; Package["dbndns"],<br />
        }  </br></br></br>

file { "/etc/service/tinydns":<br />
                ensure =&gt; "link",<br />
                target =&gt; "/etc/tinydns",<br />
                require =&gt; [ File['/etc/service'],<br />
                                        Exec["configure-tinydns"],<br />
                                ],<br />
        }  </br></br></br></br></br></br>

file { "/etc/service/dnscache":<br />
                ensure =&gt; "link",<br />
                target =&gt; "/etc/dnscache",<br />
                require =&gt; [  File['/etc/service'],<br />
                                        Exec["configure-dnscache"]<br />
                                ],<br />
        }  </br></br></br></br></br></br>

}

<hr />

[1]
http://higherlogic.com.au/2011/djbdns-on-ubuntu-10-04-server-migration-from-bind-and-zone-transfers-to-secondaries-bind/<br />
[2] https://github.com/shupp/VegaDNS/blob/master/update-data.sh</br>



</hr></hr>
