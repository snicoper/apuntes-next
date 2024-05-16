---
sidebar_position: 1
title: "Swashbuckle"
id: swashbuckle
---

## Instalación

Añadir paquete `Swashbuckle.AspNetCore` en `src/WebApi/WebApi.csproj`

## DependencyInjection

Editar `src/WebApi/DependencyInjection.cs` y añadir.

```cs
// OpenApi.
services.AddSwaggerGen(
    options =>
    {
        options.SwaggerDoc(
            "v1",
            new OpenApiInfo { Version = "v1", Title = "CleanArchitecture API", Description = "An ASP.NET Core Web API" });

        options.AddSecurityDefinition(
            "Bearer",
            new OpenApiSecurityScheme
            {
                In = ParameterLocation.Header,
                Description = "Please insert JWT with Bearer into field",
                Name = "Authorization",
                Type = SecuritySchemeType.ApiKey
            });

        var xmlFilename = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
        options.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, xmlFilename));
    });
```

## Program

Editar `src/WebApi/Program.cs` y añadir.

```cs
// Initialise Database in development.
if (app.Environment.IsDevelopment())
{
    await app.InitialiseDatabaseAsync();

    app.UseSwagger(); // <-
    app.UseSwaggerUI( // <-
        options => // <-
        { // <-
            options.SwaggerEndpoint("/swagger/v1/swagger.json", "v1"); // <-
            options.RoutePrefix = string.Empty; // <-
        }); // <-
}
else
{
    app.UseHsts();
}
```
