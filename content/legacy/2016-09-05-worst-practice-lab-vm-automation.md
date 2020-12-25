---

title: Worst Practice Lab VM Automation
author: ytjohn
date: 2016-09-05 21:50:36

layout: post

slug: worst-practice-lab-vm-automation
tags: [homelab]
---
# Worst Practice Lab VM Automation

I've started the process of switching my lab over from unmanaged to ansible. I've used Puppet and Salt quite extensively through work, but after a handful of false starts with the lab, I think ansible is the way to go.g his is a series of what
many (including myself) would consider "worst practices", but are more along the lines of "rapid iteration". The goal here
is to get something working in a short period of time, without spending hours, days, or weeks researching best practices.
This is instead something someone can put together on a Sunday afternoon, in between chasing after a 3 year old.

These are a handful of manual steps, each of which could be easily automated once you determine your "starting point".

*Background:* When I clone a VM in proxmox, it comes up with the hostname "xenial-template". I should be able to do something like I do with cloud-init under kvm, but I haven't gotten that far under the proxmox setup. Additionally, these hosts are not in dns until they are entered into the freeipa server. Joining a client to IPA will automatically create the entry. So the first thing I need to do to any VM is to set the hostname, fqdn, and then register it with IPA. My template
has a user called "yourtech", which I can use to login and configure the VM.

First, create an ansible vault password file: `echo secret&gt; ~/.vault_pass.txt`. Next, create an and inventory directory and setup an encrypted `group_vars/all`.

```
mkdir -p inventory/group_vars
touch inventory/group_vars/all
```

Add some common variables to `all`:

```
---
ansible_ssh_user: yourtech
ansible_ssh_pass: secret
ansible_sudo_pass: secret
freeipaclient_server: dc01.lab.ytnoc.net
freeipaclient_domain: lab.ytnoc.net
freeipaclient_enroll_user: admin
freeipaclient_enroll_pass: supersecret
```

Then encrypt it: `ansible-vault --vault-password-file=~/.vault_pass.txt encrypt inventory/group_vars/all`

## Generate inventory files.

With the following script, I can run `./add-new.sh example 192.168.0.121`. If ansible failes, then I need to
troubleshoot. A better approach would be to add these entries into a singular inventory file, or better yet,
a database, providing a constantly updated and dynamic inventory. Put that on the later pile.

```
#!/usr/bin/env bash

NEWNAME=$1
IP=$2
DOMAIN=lab.ytnoc.net
FQDN="${NEWNAME}.${DOMAIN}"
ANSIBLE_VAULT_PASSFILE=~/.vault_pass.txt
BASEDIR=~/projects/ytlab/inventory
FILENAME="${BASEDIR}/${NEWNAME}"
LINE="${FQDN} ansible_host=${IP}"

export ANSIBLE_HOST_KEY_CHECKING=False

echo ${LINE} &gt; ${FILENAME}

echo "Removing any prior host keys"
ssh-keygen -R ${NEWNAME}
ssh-keygen -R ${FQDN}
ssh-keygen -R ${IP}

echo "${FILENAME} created, testing"
ansible --vault-password-file ${ANSIBLE_VAULT_PASSFILE} -i ${FILENAME} ${FQDN} -m ping -vvvv
```

# Let's go to work.

At this point, I should have a working inventory file for a single host and I've validated that ansible can
connect. Granted, I haven't tested `sudo`, but in my situation, I'm pretty sure that will work. But I haven't
actually done anything with the VM. It's still just this default template.

## FQDN

Ansible provides a module to set the hostname, but does not modify `/etc/hosts` to get the FQDN resolving. As with
many things, I'm not the first to encounter this, so I found a premade role [holms/ansible-fqdn](https://github.com/holms/ansible-fqdn.git).

```
mkdir roles
cd roles
git clone https://github.com/holms/ansible-fqdn.git fqdn
```

This role will read `inventory_hostname` for fqdn, and `inventory_hostname_short` for hostname. You can override
this, but these are perfect defaults based on my script above.

## FreeIPA

Once again, we're saved by the Internet. [alvaroaleman/ansible-freeipa-client](https://github.com/alvaroaleman/ansible-freeipa-client.git) is an already designed role that installs the necessary freeipa packages and runs the
ipa-join commands.

```
# assuming still in roles
git clone https://github.com/alvaroaleman/ansible-freeipa-client.git freeipa
```

The values this module needs just happens to perfectly match the freeipa_* variables I put in my `all` file earlier. I
think that's just amazing luck.

## Make a playbook.

I call mine `bootstrap.yml`.

```
---
- hosts: all
become: yes
roles:
- fqdn
- freeipa
```

## Execute

Let's run our playbook against host "pgdb02"

`ansible-playbook -i inventory/pgdb02 --vault-password-file=~/.vault_pass.txt bootstrap.yml`

*Output:*
```
ytjohn@corp5510l:~/projects/ytlab$ ansible-playbook -i inventory/pgdb02 --vault-password-file=~/.vault_pass.txt base.yml

PLAY ***************************************************************************

TASK [setup] *******************************************************************
ok: [pgdb02.lab.ytnoc.net]

TASK [fqdn : fqdn | Configure Debian] ******************************************

TASK [fqdn : fqdn | Configure Redhat] ******************************************
skipping: [pgdb02.lab.ytnoc.net]

TASK [fqdn : fqdn | Configure Linux] *******************************************
included: /home/ytjohn/projects/ytlab/roles/fqdn/tasks/linux.yml for pgdb02.lab.ytnoc.net

TASK [fqdn : Set Hostname with hostname command] *******************************
changed: [pgdb02.lab.ytnoc.net]

TASK [fqdn : Re-gather facts] **************************************************
ok: [pgdb02.lab.ytnoc.net]

TASK [fqdn : Build hosts file (backups will be made)] **************************
changed: [pgdb02.lab.ytnoc.net]

TASK [fqdn : restart hostname] *************************************************
ok: [pgdb02.lab.ytnoc.net]

TASK [fqdn : fqdn | Configure Windows] *****************************************
skipping: [pgdb02.lab.ytnoc.net]

TASK [freeipa : Assert supported distribution] *********************************
ok: [pgdb02.lab.ytnoc.net]

TASK [freeipa : Assert required variables] *************************************
ok: [pgdb02.lab.ytnoc.net]

TASK [freeipa : Import variables] **********************************************
ok: [pgdb02.lab.ytnoc.net]

TASK [freeipa : Set DNS server] ************************************************
skipping: [pgdb02.lab.ytnoc.net]

TASK [freeipa : Update apt cache] **********************************************
ok: [pgdb02.lab.ytnoc.net]

TASK [freeipa : Install required packages] *************************************
changed: [pgdb02.lab.ytnoc.net] =&gt; (item=[u'freeipa-client', u'dnsutils'])

TASK [freeipa : Check if host is enrolled] *************************************
ok: [pgdb02.lab.ytnoc.net]

TASK [freeipa : Enroll host in domain] *****************************************
changed: [pgdb02.lab.ytnoc.net]

TASK [freeipa : Include Ubuntu specific tasks] *********************************
included: /home/ytjohn/projects/ytlab/roles/freeipa/tasks/ubuntu.yml for pgdb02.lab.ytnoc.net

TASK [freeipa : Enable mkhomedir] **********************************************
changed: [pgdb02.lab.ytnoc.net]

TASK [freeipa : Enable sssd sudo functionality] ********************************
changed: [pgdb02.lab.ytnoc.net]

RUNNING HANDLER [freeipa : restart sssd] ***************************************
changed: [pgdb02.lab.ytnoc.net]

RUNNING HANDLER [freeipa : restart ssh] ****************************************
changed: [pgdb02.lab.ytnoc.net]

PLAY RECAP *********************************************************************
pgdb02.lab.ytnoc.net : ok=18 changed=8 unreachable=0 failed=0
```

# Recap

Essentially, we created a rather basic inventory generator script, we encrypted some
credentials into a variables file using ansible-vault, and we downloaded some roles
&quot;off the shelf&quot; and executed them both with a single &quot;bootstrap&quot; playbook.

If I was doing this for work, I would first create at least one Vagrant VM and work through
an entire development cycle. I would probably rewrite these roles I downloaded to make them
more flexible and variable driven.

In case you got lost where these files go:

```
.
├── add-new.sh
├── bootstrap.yml
├── inventory
│   ├── group_vars
│   │   ├── all
│   ├── pgdb01
│   ├── pgdb02
│   └── sstorm01
└── roles
├── fqdn
└── freeipa
```
