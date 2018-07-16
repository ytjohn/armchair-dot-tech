---
title: ReImagining My Homelab
author: ytjohn
date: 2018-07-16 12:35:40
layout: post
slug: homelab-reimagined
tags: homelab, ansible, reimagine
---

For years I've run servers, raspberry pis, NAS, IP cameras and managed switches out of my home. For the most part, it has been unmanaged. I've moved some of the deployment and automation under ansible roles, and I've made half-hearted attempts at adding monitoring and documentation, but nothing too permanent. I recently acquired a new server in the form of a Dell R710 and I've decided to use this system to sort of redesign and re-imagine my homelab from the ground up.

### Goals

Here is what I'd like to see when I finish:

* Monitoring of every device and service with alerts going to my phone via Telegram
* Monitoring of internet connectivity
* Monitoring of my "production" websites and servers. 
* Plex and associated apps fully managed in containers
* IP cameras accessible using normal modern web browsers (ovnif to ffmpeg translation layer)
* I have a 6-in-1 temp/humidity/light/motion sensor logging to a raspberry pi. I want to send this to an MQTT queue and graph it in grafana.
* Additionally graph outdoor temperatures, both from a service and a yet to be installed outdoor weather station.
* I've written some code to poll my WaterFurnace for power usage. Get that into a TSDB and graph it. 
* Backups of any and all databases, including the TSDB.

First things first, everytihng will be managed by ansible. From installing docker to deploying and configuring the containers. Not only will it maintain the racked servers, but also bring any raspberry Pis on the network under control.

Secondly, I'm going to be putting everything possible into containers right on the bare metal. I've been learning kubernetes for work, but I'm not quite ready to make that dive for home. Let's consider Docker a transistional step and maybe later down the road, I'll convert over to kubernetes and helm charts. Ansible is really good at managing docker and is a [better substitute for docker-compose](https://www.ansible.com/blog/six-ways-ansible-makes-docker-compose-better). 

Third, I'm going to start (shouldn't that be a first point then?) with a set of common services: time series database, messaging protocol, graphing, and alertings. This will make up my monitoring and recording stack that any other tool I add on later can utilize. 

### Commons

These common services are not to be confused with core services like ntp and dns. If they go down, I lose visibility but not functionality.

As this is a homelab, I'm not worried about scaling. I have 4 "servers" in my homelab. I'm going to be putting all of my "commons" in one basket, the R710. My media server will remain separate, and the other 2 servers I am migrating away from, though they will be kept on for experimental purposes. 

* [InfluxDB](https://www.influxdata.com/time-series-platform/influxdb/) this will be the time series database. 
* [Promethues](https://prometheus.io/) for monitoring and alerting. Prometheus can perform active checks against all my systems and record any metrics. It can also alert based on those metrics. Prometheus will also utilize InfluxDB for storage of long-term metrics. 
* [MQTT](http://mqtt.org/) is a machine-to-machine pub/sub messaging transport. This has pretty much become the defacto standard for IoT devices such as temperature sensors. At this point, I'm not sure which broker I'm going to use, but I'm leaning towards either [vernemq](https://vernemq.com/) or [rabbitmq](https://www.rabbitmq.com/mqtt.html). Using RabbitMQ would additionally provide the AMQP message bus as well, but I'm not sure if I will actually need that.
* [Grafana](https://grafana.com/) for graphing. It can read from InfluxDB or Prometheus and there is a lot of community support for building beautiful dashboards here. 

With these common services in place before anything else, I will know exactly how to point or configure all the additional services that I add down the road. There are adapters that will feed MQTT messages into InfluxDB, and its constantly getting easier to get prometheus format metrics for almost any kind of service or application.

## First Prep

When I setup a new VM or bare metal machine of which I am the primary user, I have this "firstprep" playbook that I run ad-hoc. It sets my primary key, gives me passwordless sudo, and disables root+password access via ssh. *This is a dangerous playbook.* Before I run it, I log into the machine in question, start a screen session, then sudo to root. If I break anything here, I should be able to recover from that screen session. 

This is really a one-shot script that I override things with environment variables just to get a hand installed machine to a state where future ansible runs are much easier to manage.

```
# standard run when 
ansible-playbook -i inveigh.lab.ytnoc.net, plays/firstprep.yml -u ytjohn --ask-become-pass

# run that disables the user "pi", common on raspbian.
ansible-playbook -i inveigh.lab.ytnoc.net, plays/firstprep.yml -u ytjohn -e remove_pi=true
```

And here is that playbook, with vars defined at the top.

```
---
# firstprep is designed to be generic enough to use on and off lab

- hosts: all
  become: true

  vars:
    primary_user: ytjohn
    primary_user_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCneBn1KKh0vQMO2qFOoCulOjVuv87FqCtuw+gPLjnIikHfanFZLb6Ei6+BC1DxP+OhGbiDkSwWWYH7rAXzx6H++2mkuE4MhZofXUTy1gv7CbgVlLem/nCywU8L//3o6ab+Od57cpRUfj90NSXhwGlwcs5cCaClMVKjFy8SgDq0t8uzlJZkGASUUEiPWQN+whC9NQE6ve6FBcty1+DUG45pkhXmeXKTDSKGkDfnTfeFvid5ls4lAHZpNr9ilss40Z7rASpOnb4/EKnyZsvTWUuWqJkmb2hrfLqCgMoMcduADuNjkBqN5I8bBazahJ3B99bW1qZywvApcpbdI6umWzHn ytjohn@yourtech"
    install_fail2ban: true
    disable_root_login: true
    disable_password_login: true
    remove_pi: false

  tasks:

    - name: Add primary user
      user:
        name: "{{ primary_user }}"
        system: yes
        state: present
        shell: /bin/bash
        home: "/home/{{ primary_user }}"

    - name: Allow primary user to have passwordless sudo
      lineinfile:
        dest: /etc/sudoers
        state: present
        regexp: "'^{{ primary_user }}'"
        line: "{{ primary_user }} ALL=(ALL) NOPASSWD: ALL"

    - name: Set up authorized keys for primary user
      authorized_key: 
        user: ytjohn 
        key: "{{ primary_user_key }}"

    - name: intall fail2ban
      package:
        name: fail2ban
        state: present
      when: install_fail2ban

    - name: Disable remote root login
      lineinfile:
        dest: "{{ sshd_config }}"
        regexp: "^#?PermitRootLogin"
        line: "PermitRootLogin no"
      when: disable_root_login
      notify: restart ssh

    - name: Disable password login
      lineinfile:
        dest: "{{ sshd_config }}"
        regexp: "^#?PasswordAuthentication"
        line: "PasswordAuthentication no"
      when: disable_password_login
      notify: restart ssh

    - name: Remove pi user
      user:
        name: pi
        state: absent
      when: remove_pi

  handlers:
    - name: restart ssh
      service:
        name: ssh
        state: restarted
```

## Install docker and commons

For our larger setups at work, I'm always seeking out roles in ansible galaxy or making roles to be shared across teams. However, for a homelab, I decided to focus on simplicity and make a playbook specifically for my Dell R710, which I named "inveigh" (it's actually the rage of YourTech).  So this is a one-server
playbook with variables defined at the top. I want anyone to be able to look at it and see exactly what is happening. Later, if I decide to scale this out, I might revisit rewriting this as a multi server playbook. 

This playbook installs pip, python-docker, docker-ce, adds my user (primary_user) to the docker group. After that, it creates volumes and containers for the following: influxdb, vernemq, and grafana.

This gets those services up and running. However, it does not configure any of them. I also omitted prometheus at this point because prometheus requires a lot of configuration. I may be using [this role](https://github.com/mkrakowitzer/ansible-prometheus) as long as it still works (last updated 6 months ago). 

```
---
- hosts: inveigh.lab.ytnoc.net
  become: true

  vars:
    primary_user: ytjohn

  tasks:

    - name: install pip
      apt:
        name: python-pip
        state: present
      tags:
        - docker
        - python

    - name: install docker python
      pip:
        name: docker-py
        state: present
      tags:
        - docker
        - python

    - name: docker community repo key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        id: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88
      tags: docker

    - name: docker repo
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
        state: present
        update_cache: yes
      tags: docker

    - name: intall docker
      package:
        name: docker-ce
        state: present
      tags: docker
      notify:
        - restart docker

    - name: docker service start at boot
      service:
        name: docker
        enabled: yes
      tags: docker

    - name: give primary_user access to docker
      user:
        name: "{{ primary_user }}"
        groups: docker
        append: yes
      tags: docker


    # everything below is enabling specific docker containers

    # WARNING: So I don't know why, but until I created a test volume on the host the
    # first time, creating a volume with docker_volume fails
    - name: influxdb docker volume
      docker_volume:
        name: influxdb_data
      tags:
        - containers
        - influxdb
    - name: influxdb container
      docker_container:
        name: influxdb
        image: influxdb
        restart_policy: always
        volumes:
          - influxdb_data:/var/lib/influxdb
        ports:
          - 8086:8086
      tags:
        - containers
        - influxdb

    # https://github.com/erlio/docker-vernemq
    # docker run -p 1883:1883 -e "DOCKER_VERNEMQ_ALLOW_ANONYMOUS=on" --name vernemq1 -d erlio/docker-vernemq
    # MQTT is a stateless message queue, no need for a volume
    - name: vernemq container
      docker_container:
        name: vernemq
        image: erlio/docker-vernemq
        restart_policy: always
        ports:
          - 1883:1883
      tags:
        - containers
        - vernemq

    # GRAFANA: http://docs.grafana.org/installation/docker/
    - name: grafana volume
      docker_volume:
        name: grafana_storage
      tags:
        - containers
        - grafana
    - name: grafana container
      docker_container:
        name: grafana
        image: grafana/grafana
        restart_policy: always
        volumes:
          grafana_storage:/var/lib/grafana
        ports:
          - 3000:3000
        env:
          GF_INSTALL_PLUGINS: grafana-clock-panel,grafana-simple-json-datasource
      tags:
        - containers
        - grafana

  handlers:
    - name: restart docker
      service:
        name: docker
        state: restarted
```

