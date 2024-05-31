---
sidebar_position: 1
title: "XML"
id: xml
---

```json
{
    "csproj headers": {
        "prefix": "_csproj_namespace",
        "body": [
        "<PropertyGroup>",
        "  <RootNamespace>${0:CleanArchitecture.Application}</RootNamespace>",
        "  <AssemblyName>${0:CleanArchitecture.Application}</AssemblyName>",
        "</PropertyGroup>"
        ],
        "description": "csproj namespaces"
    }
}
```
