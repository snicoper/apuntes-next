---
sidebar_position: 1
title: "gitconfig"
id: dotfiles-gitconfig
---

```yml
[user]
  name = Salvador Nicolas
  email = snicoper@gmail.com
[color]
  ui = true
[core]
  editor = vim
  autocrlf = input
  eol = lf
[alias]
  lg = log --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr %an)%Creset' --abbrev-commit --date=relative
  co = checkout
  cm = commit
  aw = switch
  st = status
  br = branch
[merge]
  tool = meld
[diff]
  tool = meld
[mergetool]
  prompt = false
  keepBackup = false
  trustExitCode = false
  keepTemporaries = false
[push]
  default = simple
[pull]
 rebase = false
[init]
 defaultBranch = main
```
