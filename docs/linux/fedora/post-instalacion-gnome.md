---
sidebar_position: 1
title: "Post instalación Fedora Gnome"
id: post-instalacion-fedora-gnome
---

## Versión probada 40

## Desinstalar

```bash
sudo dnf remove -y rhythmbox
```

## Programas básicos

```bash
sudo dnf -y install \
    dconf-editor \
    gnome-tweaks \
    gparted \
    transmission-gtk

# Gnome extensions.
flatpak install flathub org.gnome.Extensions
```

## Firewalld

Poner por defecto ``zone=public`` y añadir la red local a ``trusted``

```bash
sudo firewall-cmd --set-default-zone=public
sudo firewall-cmd --permanent --zone=trusted --add-source=192.168.0.1/24
sudo firewall-cmd --reload
```
