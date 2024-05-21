---
sidebar_position: 1
title: 'Post instalación fedora'
id: post-instalacion-fedora
---

## Versión probada Fedora 40

## Actualizar

```bash
sudo dnf -y update
sudo hostnamectl --static set-hostname ns1.snicoper.local
```

## Programas básicos

```bash
sudo dnf -y install \
    cmake \
    cpp \
    dejavu-sans-mono-fonts \
    gcc \
    gcc-c++ \
    google-chrome-stable \
    git \
    hunspell-es \
    jetbrains-mono-fonts \
    kernel-devel \
    kernel-headers \
    p7zip \
    p7zip-plugins \
    powerline \
    powerline-fonts \
    openssl \
    unrar \
    vim \
    wget \
    zsh
```

## Fuentes

- [githubnext](https://monaspace.githubnext.com/)
- [Meslo.zip](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.0.0/Meslo.zip)

```bash
mkdir -p ~/.local/share/fonts/
cp *.otf ~/.local/share/fonts/
```

## Idiomas

```bash
sudo vim /etc/locale.conf
```

```bash
LANG=en_US.UTF-8
LC_NUMERIC=es_ES.UTF-8
LC_TIME=es_ES.UTF-8
LC_MONETARY=es_ES.UTF-8
LC_PAPER=es_ES.UTF-8
LC_MEASUREMENT=es_ES.UTF-8
LC_CTYPE=es_ES.UTF-8
LC_COLLATE=en_US.UTF-8
LC_MESSAGES=en_US.UTF-8
LC_NAME=es_ES.UTF-8
LC_ADDRESS=es_ES.UTF-8
LC_TELEPHONE=es_ES.UTF-8
LC_IDENTIFICATION=es_ES.UTF-8
```

## VSCode

[https://code.visualstudio.com/docs/setup/linux](https://code.visualstudio.com/docs/setup/linux)

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" | sudo tee /etc/yum.repos.d/vscode.repo > /dev/null

sudo dnf install code -y
```

## Flatpak

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Github Desktop.
flatpak install flathub io.github.shiftey.Desktop

# Dbeaver.
flatpak install flathub io.dbeaver.DBeaverCommunity

# Postman
flatpak install flathub com.getpostman.Postman

# Insomnia.
flatpak install flathub rest.insomnia.Insomnia
```

### Postman error al hacer login

```bash
cd ~/.var/app/com.getpostman.Postman/config/Postman/proxy

openssl req -subj '/C=US/CN=Postman Proxy' -new -newkey rsa:2048 -sha256 -days 365 -nodes -x509 -keyout postman-proxy-ca.key -out postman-proxy-ca.crt
```
