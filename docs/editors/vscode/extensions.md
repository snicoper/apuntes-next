---
sidebar_position: 1
title: "Extensions"
id: extensions
---

## Ver lista de extensiones instaladas

```bash
code --list-extensions | xargs -L 1 echo code --install-extension
```

```bash
# Editor.
code --install-extension editorconfig.editorconfig
code --install-extension henriiik.vscode-sort
code --install-extension aaron-bond.better-comments
code --install-extension ms-azuretools.vscode-docker
code --install-extension hediet.vscode-drawio

## Http client.
code --install-extension rangav.vscode-thunder-client
code --install-extension bruno-api-client.bruno
code --install-extension humao.rest-client

## AI.
code --install-extension Codeium.codeium

## Vim.
code --install-extension vscodevim.vim

## Específicos en Windows.
code --install-extension ms-vscode.powershel

# Spell.
code --install-extension streetsidesoftware.code-spell-checker
code --install-extension streetsidesoftware.code-spell-checker-spanish

# Git.
code --install-extension eamodio.gitlens
code --install-extension mhutchie.git-graph
code --install-extension pflannery.vscode-versionlens

# Linters.
code --install-extension esbenp.prettier-vscode
code --install-extension dbaeumer.vscode-eslint
code --install-extension davidanson.vscode-markdownlint
code --install-extension usernamehw.errorlens
code --install-extension sonarsource.sonarlint-vscode

# C#.
code --install-extension ms-dotnettools.csdevkit
code --install-extension ms-dotnettools.csharp
code --install-extension ms-dotnettools.vscode-dotnet-runtime
code --install-extension ms-dotnettools.vscodeintellicode-csharp
# code --install-extension pierre3.csharp-to-plantuml

# TypeScript.
code --install-extension mariusalchimavicius.json-to-ts
code --install-extension yoavbls.pretty-ts-errors
# code --install-extension AlexShen.classdiagram-ts

# Angular.
code --install-extension cyrilletuzi.angular-schematics
code --install-extension angular.ng-template
code --install-extension mikael.angular-beastcode

# Html Css
code --install-extension zignd.html-css-class-completion
code --install-extension formulahendry.auto-rename-tag
code --install-extension mrmlnc.vscode-scss
code --install-extension ms-vscode.live-server
code --install-extension kisstkondoros.vscode-gutter-preview

# Theme
code --install-extension pkief.material-icon-theme
code --install-extension beardedbear.beardedtheme

# ##############################
# Otros themes.
#
# code --install-extension beardedbear.beardedtheme # -> "workbench.colorTheme": "Bearded Theme Anthracite",
# code --install-extension EdwinSulaiman.jetbrains-rider-dark-theme
# code --install-extension enkia.tokyo-night
# code --install-extension uloco.theme-bluloco-dark
# code --install-extension liviuschera.noctis
```
