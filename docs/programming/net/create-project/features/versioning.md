---
sidebar_position: 1
title: "API Versioning"
id: api-versioning
---

## Instalación

Añadir paquete `Microsoft.AspNetCore.Mvc.Versioning` en `src/WebApi/WebApi.csproj`

Editar `src/WebApi/DependencyInjection.cs` y añadir:

```cs
// API versioning.
services.AddApiVersioning(
    options =>
    {
        options.ReportApiVersions = true;
        options.DefaultApiVersion = new ApiVersion(1, 0);
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.ApiVersionReader = new UrlSegmentApiVersionReader();
    });
```

Editar `src/WebApi/Controllers/HelloController.cs` y añadir `{version:apiVersion}` en el `Route`

```cs
[Route("api/v{version:apiVersion}/hello")]
public class HelloController : ApiControllerBase
{
}
```
