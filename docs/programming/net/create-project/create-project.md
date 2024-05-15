---
sidebar_position: 1
title: "Creación de proyecto"
id: creacion-proyecto
---

## Añadir funcionalidades a un proyecto existente

- [Funcionalidades](./funcionalidades)

## Crear un proyecto de cero

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

- [Solution items](./solution-items/solution-items.md)

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

Se referencia con `Domain`.

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <RootNamespace>EmployeeControl.Application</RootNamespace>
    <AssemblyName>EmployeeControl.Application</AssemblyName>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\Domain\Domain.csproj" />
  </ItemGroup>
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

Se referencia con `Application`.

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <RootNamespace>EmployeeControl.Infrastructure</RootNamespace>
    <AssemblyName>EmployeeControl.Infrastructure</AssemblyName>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\Application\Application.csproj" />
  </ItemGroup>
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

Se referencia con `Infrastructure` y `Application`.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <RootNamespace>CleanArchitecture.WebApi</RootNamespace>
    <AssemblyName>CleanArchitecture.WebApi</AssemblyName>
    <UserSecretsId>clean-architecture-b50e9c12-c1ea-4af4-87cf-88e9343ed6f8</UserSecretsId>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\Application\Application.csproj" />
    <ProjectReference Include="..\Infrastructure\Infrastructure.csproj" />
  </ItemGroup>

  <ItemGroup>
    <Content Include="..\..\.dockerignore">
      <Link>.dockerignore</Link>
    </Content>
  </ItemGroup>
</Project>
```

:::note
Cambiar el **GUID** de `<UserSecretsId>clean-architecture-b50e9c12-c1ea-4af4-87cf-88e9343ed6f8</UserSecretsId>`, se puede generar uno desde [https://www.guidgenerator.com/](https://www.guidgenerator.com/)
:::

Crear `DependencyInjection.cs`

```cs
namespace CleanArchitecture.WebApi;

public static class DependencyInjection
{
    public static IServiceCollection AddWebApi(this IServiceCollection services)
    {
        services.AddControllersWithViews();

        services.AddRouting(options => { options.LowercaseUrls = true; });

        return services;
    }
}

```

Modificar `Program.cs`

```cs
using Microsoftusing CleanArchitecture.WebApi;
using EmployeeControl.Application;
using EmployeeControl.Infrastructure;

var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddApplication()
    .AddInfrastructure()
    .AddWebApi();

var app = builder.Build();

app.UseHttpsRedirection();

app.MapControllers();

app.Run();
```

## ApiControllerBase

Crear `src/WebApi/Infrastructure/ApiControllerBase.cs`.

Mas adelante se utilizara para funcionalidades globales en los **Controllers** como **MediatR**.

```cs
using Microsoft.AspNetCore.Mvc;

namespace CleanArchitecture.WebApi.Infrastructure;

[ApiController]
[Produces("application/json")]
[ProducesResponseType(StatusCodes.Status401Unauthorized)]
public class ApiControllerBase : ControllerBase
{
}
```

## HelloController

Crear `src/WebApi/Controllers/HelloController.cs` para hacer un simple test.

```cs
using CleanArchitecture.WebApi.Infrastructure;
using Microsoft.AspNetCore.Mvc;

namespace CleanArchitecture.WebApi.Controllers;

[Route("api/hello")]
public class HelloController : ApiControllerBase
{
    [HttpGet]
    public ActionResult<string> SayHello()
    {
        return "Hello .NET";
    }
}
```

Ejecutar aplicación y navegar a [https://localhost:7000/api/hello](http://localhost:5000/api/hello), (cambiar puerto)
