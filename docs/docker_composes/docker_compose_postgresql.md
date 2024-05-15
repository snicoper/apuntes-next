---
sidebar_position: 1
title: "PostgreSQL"
id: docker-compose-postgresql
---

```yml
# docker-compose -f ./docker-compose-postgres.yml up -d
version: "4.0"
services:
  db:
    image: postgres
    restart: always
    container_name: postgres-latest
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=Password44!
      - TZ=UTC
    ports:
      - "5432:5432"
    volumes:
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```
