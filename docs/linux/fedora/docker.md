---
sidebar_position: 1
title: "Docker"
id: docker
---

## Fedora 40

```bash
sudo dnf remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-selinux \
    docker-engine-selinux \
    docker-engine
```

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo groupadd docker
sudo usermod -aG docker snicoper

sudo systemctl enable docker
sudo systemctl start docker
```

```bash
sudo firewall-cmd --permanent --zone=docker --change-interface=docker0
sudo firewall-cmd --reload
```

## Docker Desktop

[https://docs.docker.com/desktop/install/fedora/](https://docs.docker.com/desktop/install/fedora/)
