---
sidebar_position: 1
title: "Añadir Serilog"
id: serilog
---

## Fuentes

- [https://github.com/serilog/serilog](https://github.com/serilog/serilog)

## Añadir al proyecto

Añadir paquete en `WebApi.sln` -> `Serilog.AspNetCore`

:::info
El paquete `Serilog.AspNetCore` sería lo mismo que añadir `Serilog`, `Serilog.Sinks.Console` y `Serilog.Sinks.File`.

Hay mas **Sinks** para otros casos de uso.
:::

## Editar appsettings.json

Editar el archivo `src/WebApi/appsettings.Development.json` y `src/WebApi/appsettings.json` y Remplazar el `Logging`por lo siguiente:

```json
"Serilog": {
    "Using": [
        "Serilog.Sinks.Console",
        "Serilog.Sinks.File"
    ],
    "MinimumLevel": {
        "Default": "Information",
        "Override": {
        "Microsoft": "Information",
        "System": "Warning"
        }
    },
    "WriteTo": [
        {
        "Name": "Console"
        },
        {
        "Name": "File",
        "Args": {
            "path": "/logs/log-development-.log",
            "rollingInterval": "Day",
            "rollOnFileSizeLimit": true,
            "formatter": "Serilog.Formatting.Compact.CompactJsonFormatter, Serilog.Formatting.Compact"
        }
        }
    ],
    "Enrich": [
        "FromLogContext",
        "WithMachineName",
        "WithThreadId"
    ]
}
```

Luego modificar valores según **Development** o **Production**.

## Program.cs

Editar `src/WebApi/Program.cs`.

```cs
builder.Host.UseSerilog(
    (context, configuration) =>
        configuration.ReadFrom.Configuration(context.Configuration));

var app = builder.Build();

app.UseSerilogRequestLogging();
```

:::info
Importante, añadir el middleware `app.UseSerilogRequestLogging();` antes de otros middlewares para que se puedan registrar los logs en la terminal del servidor web del resto de middlewares.
:::
