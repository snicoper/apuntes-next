---
sidebar_position: 1
title: "StyleCopt"
id: stylecop
---

## Fuentes

- [https://github.com/DotNetAnalyzers/StyleCopAnalyzers](https://github.com/DotNetAnalyzers/StyleCopAnalyzers)

Ver [C# language versions](https://github.com/DotNetAnalyzers/StyleCopAnalyzers?tab=readme-ov-file#c-language-versions)

## Instalación

Editar `Directory.Build.props` y añadir

```xml
<ItemGroup>
    <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.556" />
</ItemGroup>
```

Las reglas se usan las que tiene por defecto o editando el [.editorconfig](../solution-items/editorconfig.md)
