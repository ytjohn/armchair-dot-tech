---
title: "Homelab Rebuild"
date: 2019-10-28
draft: false
tags:
 - homelab
 - rebuild
 - traefik
 - docker
---

# Background

I have a homelab which has been grown organically with services installed by hand and placed randomly. I decided it's time to rebuild it "in place". Since this isn't a production system, I have a lot of latitude in taking things down and redoing it. I do have a bunch of montoring data in influxdb that I want to keep, but most of the rest can go away. I'll start first by figuring out what I have server wise.

For phsyical hardware, I have:

* Ubiquiti EdgeRouter X (ERX) for my gateway router
* Ubiquiti Unifi Mesh APs (3)
* Dell Poweredge R710 w/72GB of RAM and dual X5670 (12 cores @ 2.9Ghz).  This is called "inveigh" and I purchased it last year to consolidate my lab onto (it's all the rage).
* 2 SuperMicro servers, one running proxmox (vsdev) and the other an outdated Ubuntu+KVM (compute02). Plan to shut these down.
* A homebrew systems called "mediocore" which runs Plex and some other bits. I'll leave this up for plex, but any other services will come off of it.
* A raspberry Pi running [home-assistant.io](https://www.home-assistant.io/) and one running a weather page.

I'm running software to monitor my WaterFurnace, my weather station, ubiquiti's unifi controller, and a Factorio server. I'm going to (re)deploy this software using Ansible to Inveigh, while also making some improvements. This process will take some experimentation, so there will be a phase of deploy by hand, then codify that into ansible.

Consolidate URLS / Split DNS
-----------------------------

In my current setup, I have multiple services running in containers, each on their own port. For instance, `inveigh:3000` is grafana, `inveigh:8080` is running a [hosted instance of VS Code](https://github.com/cdr/code-server) and `weather1.local:80` is my Pi with a weather station. I'm going to consolidate all this to `https://*.svc:443`. In preparation for this, I'm setting up split-dns. On my edgerouter-x, I added a wildcard entry. ERX uses dnsmasq, so that is a format of `address=/.domain.tld/192.168.0.1`. 

```shell
set service dns forwarding option address=/.srv.example.net/10.10.1.13
```

Now, every device on my network will get the internal IP address of my server. Furthermore, I've also opened ports 80 and 443 to NAT to 10.10.1.13. For public side access, I use Cloudflare as a CDN. It also hides my public IP address. In Cloudflare's DNS, I can't setup a wildcard without going premium, but I can setup each individual <service>.srv.example.net to point to my public IP. In some future portion, I can use their API to automate creating these records. I have a static IP, but there are dynamic DNS clients for cloudflare that can update them as needed. Note that not every service will be publicly exposed. I plan to use access lists on all services and only allow public on specific services.

## Install Traefic

To receive and route web traffic, I'm going to use [Traefic](https://traefik.io/). This will run in a container on inveigh and route to my other services. One advantage of Traefic is that it look at Docker labels to reconfigure itself on the fly. It can also request certificates from Letsencrypt automatically.

Let's get traefik working. They have an ["easy to install"](https://traefik.io/#easy-to-install). Ill try this first in a temp directory. I've shut down anything listening on 80,443, and 8080.  

```shell
# download the sample file
wget -q https://raw.githubusercontent.com/containous/traefik/master/traefik.sample.toml -O traefik.toml
ytjohn@inveigh:~/tmp/traefik$ wget -q https://raw.githubusercontent.com/containous/traefik/master/traefik.sample.toml -O traefik.toml
ytjohn@inveigh:~/tmp/traefik$ docker run -p 443:443 -p 80:80 -p 8080:8080 \
    -v $PWD/traefik.toml traefik \
        --log.level=info
time="2019-10-26T22:35:34Z" level=info msg="Configuration loaded from flags."
time="2019-10-26T22:35:34Z" level=info msg="Traefik version 2.0.2 built on 2019-10-09T19:26:05Z"
time="2019-10-26T22:35:34Z" level=info msg="\nStats collection is disabled.\nHelp us improve Traefik by turning this feature on :)\nMore details on: https://docs.traefik.io/v2.0/contributing/data-collection/\n"
time="2019-10-26T22:35:35Z" level=info msg="Starting provider aggregator.ProviderAggregator {}"
```
    
I can hit it on port 80, but not 443 or 8080. Let's see if we can get this to proxy to grafana and do https. I have grafana running in a docker container named grafana and, in theory, I traefik can take the name of a container and make it a sub-domain entrypoint. I had to do some research and there were some outdated blog posts. But Traefik provides this handy quick-start user guide for Docker and Letsencrypt. I made a few modifications for my domain and cloudflare.

```yaml
version: "3.3"

services:

    traefik:
    image: "traefik"
    container_name: "traefik"
    command:
        - "--log.level=DEBUG"
        - "--api.insecure=true"
        - "--providers.docker=true"
        - "--providers.docker.exposedbydefault=false"
        - "--entrypoints.web.address=:80"
        - "--entrypoints.websecure.address=:443"
        - "--certificatesresolvers.mydnschallenge.acme.dnschallenge=true"
        - "--certificatesresolvers.mydnschallenge.acme.dnschallenge.provider=cloudflare"
        - "--certificatesresolvers.mydnschallenge.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
        - "--certificatesresolvers.mydnschallenge.acme.email=john@mailinator.com"
        - "--certificatesresolvers.mydnschallenge.acme.storage=/letsencrypt/acme.json"
    ports:
        - "80:80"
        - "443:443"
        - "8080:8080"
    environment:
        - "CLOUDFLARE_EMAIL=dono@leavethis.here"
        - "CLOUDFLARE_API_KEY=andthisneedschangedaswell"
    volumes:
        - "./letsencrypt:/letsencrypt"
        - "/var/run/docker.sock:/var/run/docker.sock:ro"

    whoami:
    image: "containous/whoami"
    container_name: "simple-service"
    labels:
        - "traefik.enable=true"
        - "traefik.http.routers.whoami.rule=Host(`whoami.srv.example.net`)"
        - "traefik.http.routers.whoami.entrypoints=websecure"
        - "traefik.http.routers.whoami.tls.certresolver=mydnschallenge"
```

In the above, I set debug logging and I'm using the acme staging server until I feel more confident. The line `providers.docker.exposedbydefault=false` means we have to add some labels to our running docker containers if we want them to get a web service. Let's go with grafana next.

Now when I setup grafana, I was doing a couple things, but there is a tool called [runlike](https://github.com/lavie/runlike) which can show you the run command to recreate the docker container. I did `runlike grafana > rungrafana.sh` and then edited the resulting file to include traefik labels. I also removed the port mapping of 3000:3000 because I expect traefik to handle that. Because I don't truly know how persistent my dashboards are, I went into the web gui and exported all of them as json as a safety.

```shell
docker run --name=grafana --hostname=grafana --user=grafana \
    --env=GF_PATHS_PLUGINS=/var/lib/grafana/plugins \
    --env=GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource \
    --env=GF_PATHS_HOME=/usr/share/grafana --env=GF_PATHS_CONFIG=/etc/grafana/grafana.ini \
    --env=GF_PATHS_PROVISIONING=/etc/grafana/provisioning --env=GF_PATHS_DATA=/var/lib/grafana \
    --env=GF_PATHS_LOGS=/var/log/grafana \
    --env=PATH=/usr/share/grafana/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    --volume=grafana_storage:/var/lib/grafana:rw --volume=/var/lib/grafana \
    --network=influxdata_influxdata  --label com.docker.compose.version="1.17.1" \
    --label com.docker.compose.service="grafana" --label com.docker.compose.oneoff="False" \
    --label com.docker.compose.project="influxdata" \
    --label com.docker.compose.config-hash="508fb6754faeac4457717d5b39c3b23f5af4a5563c8a7c57ceb363647282ff82" \
    --label com.docker.compose.container-number="1" --detach=true \
        # now we add the traefik specific bits
        --label traefik.enable="true" \
        --label traefik.http.routers.grafana.rule='Host(`grafana.srv.example.net`)' \
        --label traefik.http.routers.grafana.entrypoints="websecure" \
        --label traefik.http.routers.grafana.tls.certresolver="mydnschallenge" \
    grafana/grafana
```

I then killed my grafana container, removed it, and ran that rungrafana script above.

```shell
ytjohn@inveigh:~/influxdata$ docker stop grafana; docker rm grafana
grafana
grafana
ytjohn@inveigh:~/influxdata$ ./grafana.sh
a37d43b18dc70c63529ad655755253174c919cef86d728fcd3e895bc8065308b
```

Then I checked traefik to see what happened. I ran `docker logs traefic |grep grafana`.

It saw the event in the provider:

```logs
time="2019-10-27T00:40:09Z" level=debug msg="Provider event received {Status:start ID:2841d61cb4154142aa6e7445cc46d475bad73ed56e7955bbde0704bd4dbc60bf From:grafana/grafana Type:container Action:start Actor:{ID:2841d61cb4154142aa6e7445cc46d475bad73ed56e7955bbde0704bd4dbc60bf Attributes:map[com.docker.compose.config-hash:508fb6754faeac4457717d5b39c3b23f5af4a5563c8a7c57ceb363647282ff82 com.docker.compose.container-number:1 com.docker.compose.oneoff:False com.docker.compose.project:influxdata com.docker.compose.service:grafana com.docker.compose.version:1.17.1 image:grafana/grafana name:grafana traefik.enable:true traefik.http.routers.grafana.rule:Host(`grafana.srv.example.net`)
```

And then it grabbed a certificate.

```shell
time="2019-10-27T00:40:09Z" level=debug msg="Domains [\"grafana.srv.example.net\"] need ACME certificates generation for domains \"grafana.srv.example.net\"." providerName=mydnschallenge.acme rule="Host(`grafana.srv.example.net`)" routerName=grafana
....
time="2019-10-27T00:40:29Z" level=debug msg="Adding certificate for domain(s) grafana.srv.example.net"
```

But I couldn't reach it. And this was because I failed to put traefik and grafana on the same network. So let's do that. Let's create a network called `traefik`.

```shell
docker network create traefik
67b11d8c6ca0623d889ead5b422e2102a70786786322e8573fce36155360be06
```

I add a `networks [traffic]` to both services in the docker-compose and add the following to the bottom:

```yaml
networks:
  traefik:
    external: true
```

For good measure, I `docker-compose stop; docker-compose rm; docker-compose up -d`. Then I added a `--network traefik` to the grafana.sh command and recreated that container. After a few moments, everything worked and I could access https://grafana.srv.example.net.  Success.

For the next step, I went into traefik's docker-compose.yaml file and removed the line for staging (`- "--certificatesresolvers.mydnschallenge.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"`). Now I can get production certificates. Since I already have the one for grafana and whoami from the staging, I need to stop traefik completely, and delete my local acme.json file.

```shell
docker-compose stop; docker-compose rm
sudo rm letsencrypt/acme.json
docker-compose up -d
```

Once that is done, we should be in business for real-world production certificates for the https://whoami.srv.example.net and https://grafana.srv.example.net.  

## Ansible

I said at the beginning that I would do all this in Ansible. But everything so far has been manually done. When I do the next part, we'll blow away our traefik containers, the grafana container, and the traefik network we created. Then we'll set up Ansible to recreate them. After that, we'll probably go right into recreating the rest of my InfluxData stack (influxdb, kapacitor, chronos).




