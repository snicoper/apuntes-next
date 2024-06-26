---
sidebar_position: 1
title: "Oh my posh"
id: oh-my-posh
---

## Fuentes

[https://ohmyposh.dev/](https://ohmyposh.dev/)

## Instalar Oh my posh

```bash
winget install JanDeDobbeleer.OhMyPosh -s winget
```

## Instalar fuentes

Abrir **PowerShell** como administrador, instalar **Meslo**, **JetBrainsMono**.

```bash
oh-my-posh font install
```

`Settings` -> `PowerShell` -> `Appearance Font Face`: `JetBrainsMono Nerd Font`

## Instalar PSReadLine

Iconos al hacer ls

```bash
Install-Module PSReadLine -AllowPrerelease -Force
```

## Instalar Terminal-Icons

Abrir **PowerShell** como usuario.

```bash
Install-Module -Name Terminal-Icons -Repository PSGallery
```

## Auto completado Git

[https://www.rodyvansambeek.com/blog/git-autocomplete-powershell-magic](https://www.rodyvansambeek.com/blog/git-autocomplete-powershell-magic)

```bash
Install-Module posh-git -Scope CurrentUser
```

## PROFILE

```bash
code $PROFILE
```

```ps
# PSReadLine.
Import-Module PSReadLine

Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle ListView
Set-PSReadLineOption -EditMode Windows

# Terminal-Icons.
Import-Module Terminal-Icons

# Git.
Import-Module posh-git

# oh-my-posh.
# oh-my-posh init pwsh | Invoke-Expression
oh-my-posh init pwsh --config 'C:\Users\snicoper\AppData\Local\Programs\oh-my-posh\themes\atomic.omp.json' | Invoke-Expression
```
