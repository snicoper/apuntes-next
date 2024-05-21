---
sidebar_position: 1
title: "Settings"
id: settings
---

```json
{
  // Editor.
  "editor.fontFamily": "'Monaspace Neon', monospace",
  "editor.fontSize": 18,
  "editor.minimap.enabled": false,
  "editor.lineHeight": 27,
  "editor.mouseWheelScrollSensitivity": 2,
  "editor.suggestSelection": "recentlyUsed",
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.formatOnType": true,
  "editor.rulers": [119, 129],
  "editor.tabSize": 2,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.guides.indentation": true,
  "editor.foldingImportsByDefault": true,
  "editor.autoClosingBrackets": "always",
  "editor.linkedEditing": true,
  "editor.renderWhitespace": "none",
  "editor.renderLineHighlight": "none",
  "editor.fontLigatures": false,
  "editor.codeActionsOnSave": {
    "source.fixAll": "explicit",
    "source.fixAll.eslint": "explicit",
    "source.fixAll.tslint": "explicit",
    "source.fixAll.stylelint": "explicit",
    "source.organizeImports": "explicit"
  },
  // Terminal.
  "terminal.integrated.fontFamily": "MesloLGM Nerd Font",
  "terminal.integrated.fontSize": 16,
  // Prettier.
  "prettier.singleQuote": true,
  "prettier.printWidth": 120,
  "prettier.trailingComma": "none",
  // Languages.
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.preferences.quoteStyle": "single",
  "javascript.preferences.quoteStyle": "single",
  "typescript.format.insertSpaceAfterOpeningAndBeforeClosingTemplateStringBraces": true,
  "javascript.format.insertSpaceAfterOpeningAndBeforeClosingTemplateStringBraces": true,
  "typescript.suggest.completeFunctionCalls": true,
  "javascript.suggest.completeFunctionCalls": true,
  "[python]": {
    "editor.rulers": [79, 99],
    "editor.tabSize": 4
  },
  "[csharp]": {
    "editor.defaultFormatter": "ms-dotnettools.csharp",
    "editor.formatOnType": true,
    "editor.tabSize": 4
  },
  "[markdown]": {
    "editor.wordWrap": "on",
    "editor.tabSize": 4,
    "editor.quickSuggestions": {
      "comments": "on",
      "strings": "on",
      "other": "on"
    },
    "editor.formatOnSave": false
  },
  // Workbench.
  "workbench.colorTheme": "Bearded Theme Anthracite",
  "workbench.iconTheme": "material-icon-theme",
  "workbench.startupEditor": "none",
  "workbench.editor.tabActionCloseVisibility": false,
  "workbench.layoutControl.enabled": false,
  "workbench.colorCustomizations": {
    "activityBarBadge.background": "#9c27b0",
    "activityBarBadge.foreground": "#ffffff",
    "tab.activeBorder": "#9c27b0"
  },
  // Security.
  "security.workspace.trust.untrustedFiles": "open",
  // breadcrumbs.
  "breadcrumbs.enabled": false,
  // Files.
  "files.trimTrailingWhitespace": true,
  "files.trimFinalNewlines": true,
  "files.insertFinalNewline": true,
  "files.encoding": "utf8",
  "files.eol": "\n",
  // Explorer.
  "explorer.autoReveal": false,
  "explorer.compactFolders": false,
  // Window.
  "window.title": ".",
  "window.commandCenter": false,
  "window.titleBarStyle": "custom",
  // omnisharp.
  "omnisharp.useEditorFormattingSettings": true,
  "omnisharp.organizeImportsOnFormat": true,
  "omnisharp.enableAsyncCompletion": true,
  // Emmet.
  "emmet.includeLanguages": {
    "razor": "html",
    "aspnetcorerazor": "html"
  },
  // Sort.
  "sort.locale": "es-ES",
  // Git.
  "git.enableSmartCommit": true,
  "git.autofetch": true,
  "git.confirmSync": false,
  // Code Spell Checker.
  "cSpell.enabled": true,
  "cSpell.diagnosticLevel": "Hint",
  "cSpell.language": "en,es-ES",
  "cSpell.ignorePaths": [
    ".vscode",
    "**/.config/**",
    "**/.git/**",
    "**/.tox/**",
    "**/dist/**",
    "**/node_modules/**",
    "**/vscode-extension/**",
    "typings"
  ],
  "cSpell.userWords": [],
  // better-comments.
  "better-comments.multilineComments": true,
  "better-comments.tags": [
    {
      "tag": "fixme:",
      "color": "#FF2D00",
      "strikethrough": false,
      "underline": false,
      "backgroundColor": "transparent",
      "bold": false,
      "italic": false
    },
    {
      "tag": "?",
      "color": "#3498DB",
      "strikethrough": false,
      "underline": false,
      "backgroundColor": "transparent",
      "bold": false,
      "italic": false
    },
    {
      "tag": "//",
      "color": "#474747",
      "strikethrough": true,
      "underline": false,
      "backgroundColor": "transparent",
      "bold": false,
      "italic": false
    },
    {
      "tag": "todo:",
      "color": "#FF8C00",
      "strikethrough": false,
      "underline": false,
      "backgroundColor": "transparent",
      "bold": false,
      "italic": false
    },
    {
      "tag": "*",
      "color": "#98C379",
      "strikethrough": false,
      "underline": false,
      "backgroundColor": "transparent",
      "bold": false,
      "italic": false
    }
  ],
  // Sonarlint.
  "sonarlint.rules": {
    "css:S4649": {
      "level": "off"
    },
    "Web:S5256": {
      "level": "off"
    },
    "Web:TableWithoutCaptionCheck": {
      "level": "off"
    }
  },
  "inference.endpoint": "http://localhost:11434",
  "codeium.enableConfig": {
    "*": true,
    "markdown": true,
    "plaintext": true
  }
}
```
