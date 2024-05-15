---
sidebar_position: 1
title: "Creación de proyecto"
id: creacion-proyecto
---

Crear solución vacía `CleanArchitecture`

Crear Solution Folders `src`, `tests` y `Solution items`

Crear `src`, `tests` en la raíz del proyecto.

```bash
mkdir src tests
```

Crear el repositorio Git.

```bash
git init .
```

## Copiar archivos base

Copiar los siguientes archivos en la raíz del proyecto y añadirlos en `Solution items`.

- [Directory.Build.props](./solution_items/directory_build_props.md)
- [EditorConfig](./solution_items/editorconfig.md)
- [Gitattributes](./solution_items/gitattrubutes.md)
- [Gitignore](./solution_items/gitignore.md)
- [Global](./solution_items/global_json.md)

## Domain

Crear proyecto `Class Library` en `src` (en el solution folder y físicamente) **Domain** y editar `Domain.csproj`

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <RootNamespace>EmployeeControl.Domain</RootNamespace>
    <AssemblyName>EmployeeControl.Domain</AssemblyName>
  </PropertyGroup>
</Project>
```

## Application

Crear proyecto `Class Library` en `src` (en el solution folder y físicamente) **Application** y editar `Application.csproj`

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <RootNamespace>EmployeeControl.Application</RootNamespace>
    <AssemblyName>EmployeeControl.Application</AssemblyName>
  </PropertyGroup>
</Project>
```

Añadir `Microsoft.Extensions.DependencyInjection.Abstractions` en la capa **Application**

Crear `DependencyInjection.cs`

```cs
using Microsoft.Extensions.DependencyInjection;

namespace EmployeeControl.Application;

public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        return services;
    }
}
```

## Infrastructure

Crear proyecto `Class Library` en `src` (en el solution folder y físicamente) **Infrastructure** y editar `Infrastructure.csproj`

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <RootNamespace>EmployeeControl.Infrastructure</RootNamespace>
    <AssemblyName>EmployeeControl.Infrastructure</AssemblyName>
  </PropertyGroup>
</Project>
```

Crear `DependencyInjection.cs`

```cs
using Microsoft.Extensions.DependencyInjection;

namespace EmployeeControl.Infrastructure;

public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(this IServiceCollection services)
    {
        return services;
    }
}
```

## WebApi

Crear proyecto `Web Empty` en `src` (en el solution folder y físicamente) **WebApi** y editar `WebApi.csproj`

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <RootNamespace>EmployeeControl.WebApi</RootNamespace>
    <AssemblyName>EmployeeControl.WebApi</AssemblyName>
  </PropertyGroup>
</Project>
```

Crear `DependencyInjection.cs`

```cs
using Microsoft.Extensions.DependencyInjection;

namespace EmployeeControl.WebApi;

public static class DependencyInjection
{
    public static IServiceCollection AddWebApi(this IServiceCollection services)
    {
        return services;
    }
}
```
