---
sidebar_position: 1
title: "Scrutor"
id: scrutor
---

## Instalación

Añadir `Scrutor` en el proyecto `src/Application/Application.csproj`

## DependencyInjection

Editar `DependencyInjection` de los proyectos `WebApi`, `Infrastructure` y `Application` y añadir

```cs
// Scrutor.
services.Scan(
    scan =>
        scan.FromCallingAssembly()
            .AddClasses(classes => classes.Where(type => type.Name.EndsWith("Service")))
            .AsImplementedInterfaces()
            .WithTransientLifetime());
```
