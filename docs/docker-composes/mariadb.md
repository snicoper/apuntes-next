---
sidebar_position: 1
title: "Maria DB"
---

```yml
version: "3.9"
services:
  db:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: Password4!
      # MYSQL_DATABASE: mydatabase
      MYSQL_USER: snicoper
      MYSQL_PASSWORD: Password4!
    volumes:
      - data:/var/lib/mysql
    ports:
      - "3306:3306"
volumes:
  data:
```
