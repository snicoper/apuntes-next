---
sidebar_position: 1
title: "docker-compose.yml"
id: docker-compose-yml
---

Crear archivo `Directory.Build.props` en la raíz del proyecto.

```yml
version: '3'
services:
  redis:
    image: redis:6.2.5services:
  webapi:
    image: webapi
    build:
      context: .
      dockerfile: src/WebApi/Dockerfile

```
