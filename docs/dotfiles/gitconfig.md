---
sidebar_position: 1
title: "gitconfig"
id: gitconfig
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
  sw = switch
  st = status
  br = branch
[diff]
  tool = default-difftool
[difftool "default-difftool"]
  cmd = code --wait --diff $LOCAL $REMOTE
[merge]
  tool = default-mergetool
[mergetool "default-mergetool"]
  cmd = code --wait $MERGED
[push]
  default = simple
[pull]
  rebase = false
[init]
  defaultBranch = main
```
