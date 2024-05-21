---
sidebar_position: 1
title: "Key SSH"
id: key-ssh
---

## Generar una clave SSH

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Copiar la clave SSH del archivo `.ssh/id_ed25519.pub` y añadirla en **GitLab/GitHub**

Crear un archivo de configuración `.ssh/config`.

```bash
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_ed25519

Host gitlab.com
    HostName gitlab.com
    IdentityFile ~/.ssh/id_ed25519
```
