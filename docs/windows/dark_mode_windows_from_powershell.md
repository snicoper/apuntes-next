---
sidebar_position: 1
title: "Poner Dark/Light mode desde PowerShell"
id: windows-dark-mode-windows-from-powershell
---

## Dark

```bash
Set-ItemProperty -Path HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Themes\Personalize -Name AppsUseLightTheme -Value 0
```

## Light

```bash
Set-ItemProperty -Path HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Themes\Personalize -Name AppsUseLightTheme -Value 1
```
