---
sidebar_position: 1
title: "Cors"
id: cors
---

## Configuración

Editar `src/WebApi/DependencyInjection.cs` y añadir:

```cs
// Cors.
services.AddCors(
    options =>
    {
        options.AddPolicy(
            "DefaultCors",
            builder =>
            {
                builder
                    .WithOrigins("http://localhost:4200")
                    .AllowAnyHeader()
                    .AllowAnyMethod()
                    .AllowCredentials();
            });
    });
```

Editar `src/WebApi/Program.cs` y añadir:

```cs
app.UseRequestLocalization();
```
