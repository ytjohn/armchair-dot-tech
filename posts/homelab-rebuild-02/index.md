---
title: "Homelab Rebuild Part 2"
date: 2019-12-29
draft: false
tags:
 - homelab
 - rebuild
 - traefik
 - docker
---

I realized so much of my reading on Traefik was based on 1.7 and I was running 2.0. I jumped ahead and set my image to 2.1.1 (latest as of this post).

https://docs.traefik.io/v2.0/migration/v1-to-v2/
https://containo.us/blog/traefik-2-0-docker-101-fc2893944b9d/

The big problem I had with that blog post was that for the main traffic instance with the redirect was not working. Because I have

      - "--providers.docker.exposedbydefault=false"

I had to add these lines to the labels.

      - "traefik.enable=true"

```yaml
version: "3.3"

services:

  traefik:
    image: "traefik:v2.1.1"
    container_name: "traefik"
    command:
      - "--log.level=DEBUG"
      - "--api"
      - "--providers.docker"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.email=john@yourtech.us"
      - "--certificatesresolvers.letsencrypt.acme.storage=/acme.json"
      - "--certificatesresolvers.letsencrypt.acme.dnschallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.dnschallenge.provider=cloudflare"
    ports:
      - "80:80"
      - "443:443"
    environment:
      - "CF_API_EMAIL=john@yourtech.us"
      - "CF_API_KEY=84d0913fc36bd44ea1204d8b1b3fe02a046dd"
    volumes:
      - "/opt/traefik/acme.json:/acme.json"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`tdash.ytnoc.net`)"
      - "traefik.http.routers.traefik.service=api@internal"
      - "traefik.http.routers.traefik.tls.certresolver=letsencrypt"
      - "traefik.http.routers.traefik.tls=true"
      - "traefik.http.routers.traefik.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=admin:$$apr1$$IsuURUzx$$K6lyKZknZaCCBeJ5UmlQ30"
      # global redirect to https
      - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      # middleware redirect
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
    networks:
      - traefik

  whoami:
    image: "containous/whoami"
    container_name: "simple-service"
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.ytnoc.net`)"
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.tls=true"
      - "traefik.http.routers.whoami.tls.certresolver=letsencrypt"
      - "traefik.http.routers.whoami.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=admin:$$apr1$$IsuURUzx$$K6lyKZknZaCCBeJ5UmlQ30"

networks:
  traefik:
    external: true
```


new directory: homelab-ansible
checkin: inventory/sanitized
checkin: inventory/group_vars/sanitized

ansible-galaxy init localtraefik

