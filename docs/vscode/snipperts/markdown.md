---
sidebar_position: 1
title: "Markdown"
id: vscode-snippets-markdown
---

```json
 "Code": {
  "prefix": "_code",
  "body": [
   "```${1:label}",
   "$2",
   "```"
  ],
  "description": "Code block"
 },
 "Docusaurus_title_md": {
  "prefix": "_md_title",
  "body": [
   "---",
   "sidebar_position: 1",
   "title: \"${1:title}\"",
   "id: $2",
   "---\n",
   "$3"
  ],
  "description": "Titulo para docusaurus"
 },
}

```
