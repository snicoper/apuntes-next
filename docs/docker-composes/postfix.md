---
sidebar_position: 1
title: "Postfix"
---

```yml
# https://www.freekb.net/Article?id=3662
# https://github.com/docker-mailserver/docker-mailserver
# POR PROBAR

version: "3"
services:
  mailserver:
    image: docker.io/mailserver/docker-mailserver:latest
    container_name: mailserver
    hostname: mail.example.com
    domainname: example.com
    ports:
      - "25:25"
      - "143:143"
      - "587:587"
      - "993:993"
    volumes:
      - /usr/local/docker/mailserver/mail-data/:/var/mail/
      - /usr/local/docker/mailserver/mail-state/:/var/mail-state/
      - /usr/local/docker/mailserver/mail-logs/:/var/log/mail/
      - /usr/local/docker/mailserver/config/:/tmp/docker-mailserver/
      - /etc/localtime:/etc/localtime:ro
    environment:
      - ENABLE_SPAMASSASSIN=1
      - SPAMASSASSIN_SPAM_TO_INBOX=1
      - ENABLE_CLAMAV=1
      - ENABLE_FAIL2BAN=1
      - ENABLE_POSTGREY=1
      - ENABLE_SASLAUTHD=0
      - ONE_DIR=1
      - DMS_DEBUG=0
      - PERMIT_DOCKER=host
    cap_add:
      - NET_ADMIN
      - SYS_PTRACE
    restart: always
```
