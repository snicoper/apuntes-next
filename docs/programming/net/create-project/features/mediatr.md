---
sidebar_position: 1
title: "MediatR"
id: mediatr
---

## Fuentes

[https://github.com/jbogard/mediatr](https://github.com/jbogard/mediatr)

## Añadir a Application

Añadir los paquetes `Microsoft.Extensions.Logging.Abstractions` y `MediatR` en `src/Application/Domain.csproj`

Crear directorio `src/Application/Common/Behaviours` donde se añadirán los middlewares de **MediatR** y crear uno para manejar
las excepciones no controladas.

Crear `src/Application/Common/Behaviours/UnhandledExceptionBehaviour.cs`

```cs
using MediatR;
using Microsoft.Extensions.Logging;

namespace CleanArchitecture.Application.Common.Behaviours;

public class UnhandledExceptionBehaviour<TRequest, TResponse>(ILogger<TRequest> logger)
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : notnull
{
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        try
        {
            return await next();
        }
        catch (Exception ex)
        {
            var requestName = typeof(TRequest).Name;

            logger.LogError(
                ex,
                "CleanArchitecture Request: Unhandled Exception for Request {Name} {@Request}",
                requestName,
                request);

            throw;
        }
    }
}
```

Editar `src/Application/DependencyInjection.cs` y añadir.

```cs
// MediatR.
services.AddMediatR(
    config =>
    {
        config.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly());
        config.AddBehavior(typeof(IPipelineBehavior<,>), typeof(UnhandledExceptionBehaviour<,>));
    });
```

## ApiControllerBase

Editar `src/WebApi/Infrastructure/ApiControllerBase.cs` para añadir `MediatR.ISender`.

```cs
// ....
public class ApiControllerBase : ControllerBase
{
    private ISender? _sender;

    protected ISender Sender => _sender ??= HttpContext.RequestServices.GetRequiredService<ISender>();

    // ...
}
```

## Probar que funciona correctamente

Para probar que se ha añadido correctamente, editar `src/WebApi/Controllers/HelloController.cs` y remplazar el método
 `SayHello` por el siguiente código.

```cs
using CleanArchitecture.WebApi.Infrastructure;
using CleanArchitecture.Application.Hello.Queries;
using Microsoft.AspNetCore.Mvc;

namespace CleanArchitecture.WebApi.Controllers;

[Route("api/hello")]
public class HelloController : ApiControllerBase
{
    [HttpGet]
    public async Task<ActionResult<string>> SayHello()
    {
        return await Sender.Send(new SayHelloQuery());
    }
}
```

## SayHelloQuery

Crear `src/Application/Hello/Queries/SayHelloQuery.cs`

```cs
using MediatR;

namespace CleanArchitecture.Application.Hello.Queries;

public record SayHelloQuery : IRequest<string>;
```

## SayHelloHandler

Crear `src/Application/Hello/Queries/SayHelloHandler.cs`

```cs
using MediatR;

namespace CleanArchitecture.Application.Hello.Queries;

internal sealed class SayHelloHandler : IRequestHandler<SayHelloQuery, string>
{
    public Task<string> Handle(SayHelloQuery request, CancellationToken cancellationToken)
    {
        return Task.FromResult("Hello .NET and MediatR");
    }
}
```

## Domain BaseEvent

Crear `src/Domain/Common/BaseEvent.cs`

```cs
using MediatR;

namespace CleanArchitecture.Domain.Common;

public abstract class BaseEvent : INotification;
```
