---
sidebar_position: 1
title: "Post instalación KDE"
id: post-instalacion-kde
---

## Desinstalar

```bash
sudo dnf remove -y akregator
```

## Instalar

```bash
sudo dnf install -y transmission-qt
```

## KDEConnect

```label
firewall-cmd --zone=public --permanent --add-port=1714-1764/tcp
firewall-cmd --zone=public --permanent --add-port=1714-1764/udp
systemctl restart firewalld.service
```
