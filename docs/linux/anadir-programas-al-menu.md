---
sidebar_position: 1
title: "Añadir programas al menu"
id: anadir-programas-al-menu
---

```bash
mkdir -p ~/.local/bin ~/.local/apps
```

## Rider

```bash
gzip -d JetBrains.Rider-2024.1.3.tar.gz
tar -xvf JetBrains.Rider-2024.1.3.tar
mv JetBrains\ Rider-2024.1.3 ~/.local/apps/rider
chmod +x ~/.local/apps/rider/bin/rider.sh
ln -s ~/.local/apps/rider/bin/rider.sh ~/.local/bin/rider
```

Crear enlace .desktop

```bash
vim ~/.local/share/applications/rider.desktop
```

```bash
[Desktop Entry]
Encoding=UTF-8
Name=Rider
Comment=IDE for DotNet Core
Exec=/home/snicoper/.local/bin/rider %U
Icon=/home/snicoper/.local/apps/rider/bin/rider.png
Terminal=false
Type=Application
Categories=GNOME;Application;Development;
StartupNotify=true
```
