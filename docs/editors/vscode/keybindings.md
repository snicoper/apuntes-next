---
sidebar_position: 1
title: "Keybindings"
id: keybindings
---

```json
[
  {
    // Sort lines Ascending..
    "key": "F6",
    "command": "editor.action.sortLinesAscending",
    "when": "editorTextFocus"
  },
  {
    // Focus in current file.
    "key": "alt+f",
    "command": "workbench.files.action.showActiveFileInExplorer"
  },
  {
    // Join lines.
    "key": "ctrl+j",
    "command": "editor.action.joinLines"
  },
  {
    // Lower case.
    "key": "ctrl+alt+l",
    "command": "editor.action.transformToLowercase"
  },
  {
    // Upper case.
    "key": "ctrl+alt+u",
    "command": "editor.action.transformToUppercase"
  },
  {
    // Title case.
    "key": "ctrl+alt+t",
    "command": "editor.action.transformToTitlecase"
  },
  {
    // Missing all imports.
    "key": "alt+i",
    "command": "editor.action.sourceAction",
    "args": {
      "kind": "source.addMissingImports",
      "apply": "first"
    }
  },
  {
    // Open terminal.
    "key": "ctrl+[Semicolon]",
    "command": "workbench.action.terminal.toggleTerminal",
    "when": "terminal.active"
  },
  {
    // Open terminal.
    "key": "ctrl+`",
    "command": "-workbench.action.terminal.toggleTerminal",
    "when": "terminal.active"
  }
]
```
