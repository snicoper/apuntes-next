---
sidebar_position: 1
title: "Instalación .NET"
id: instalacion-dotnet
---

## Fuentes

* [https://docs.microsoft.com/es-es/dotnet/core/install/linux-fedora](https://docs.microsoft.com/es-es/dotnet/core/install/linux-fedora)
* [https://learn.microsoft.com/en-us/dotnet/core/install/linux-scripted-manual#manual-install](https://learn.microsoft.com/en-us/dotnet/core/install/linux-scripted-manual#manual-install)

## Probado en Fedora 40

## Versión RPM de Fedora

```bash
sudo dnf install dotnet-sdk-8.0
```

## Versión manual

[Descargar](https://dotnet.microsoft.com/en-us/download) la version a instalar.

Para el ejemplo se va a instalar las siguientes versiones:

* dotnet-sdk-8.0.300-linux-x64.tar.gz
* dotnet-sdk-9.0.100-preview.3.24204.13-linux-x64.tar.gz

Crear directorios de instalación:

```bash
mkdir ~/.dotnet
```

Editar `.zshrc` | `.bashrc`

```bash
export DOTNET_ROOT=$HOME/.dotnet
export PATH=$PATH:$DOTNET_ROOT:$DOTNET_ROOT/tools
export PATH=$PATH:$HOME/.dotnet
```

```bash
source ~/.zshrc
```

Instalación dotnet-sdk-8.0.301-linux-x64.tar.gz

```bash
cd Downloads
tar -C ~/.dotnet -xf dotnet-sdk-8.0.301-linux-x64.tar.gz
rm dotnet-sdk-8.0.301-linux-x64.tar.gz
dotnet --info
```

Instalación dotnet-sdk-9.0.100-preview.3.24204.13-linux-x64.tar.gz

```bash
cd Downloads
tar -C ~/.dotnet -xf dotnet-sdk-9.0.100-preview.3.24204.13-linux-x64.tar.gz
rm dotnet-sdk-9.0.100-preview.3.24204.13-linux-x64.tar.gz
dotnet --info
```

## Versión a utilizar en los proyectos

Crear `global.json` en la raíz del proyecto, al nivel del `.sln`

```json
{
  "sdk": {
    "version": "8.0.0",
    "rollForward": "latestFeature",
    "allowPrerelease": true
  }
}
```

```bash
dotnet --version
# 8.0.300
```

## Herramientas

```bash
dotnet tool install -g dotnet-ef
```
