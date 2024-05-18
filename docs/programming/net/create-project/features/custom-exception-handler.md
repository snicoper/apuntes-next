---
sidebar_position: 1
title: "CustomException"
id: custom-exception
---

## Excepciones

En `src/Application/Common/Exceptions` crear `ForbiddenAccessException` y en `src/Domain/Exceptions` crear `NotFoundException`.

### NotFoundException

```cs
namespace CleanArchitecture.Domain.Exceptions;

public class NotFoundException : Exception
{
    public NotFoundException()
    {
    }

    public NotFoundException(string message)
        : base(message)
    {
    }

    public NotFoundException(string message, Exception innerException)
        : base(message, innerException)
    {
    }

    public NotFoundException(string name, object key)
        : base($"Entity \"{name}\" ({key}) was not found.")
    {
    }
}
```

### ForbiddenAccessException

```cs
namespace CleanArchitecture.Application.Common.Exceptions;

public class ForbiddenAccessException : Exception;
```

### CustomExceptionHandler

Crear `src/WebApi/Infrastructure/CustomExceptionHandler.cs`

```cs
using CleanArchitecture.Application.Common.Exceptions;
using CleanArchitecture.Domain.Exceptions;
using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Mvc;

namespace CleanArchitecture.WebApi.Infrastructure;

public class CustomExceptionHandler : IExceptionHandler
{
    private readonly Dictionary<Type, Func<HttpContext, Exception, Task>> _exceptionHandlers;

    public CustomExceptionHandler()
    {
        // Register known exception types and handlers.
        _exceptionHandlers = new Dictionary<Type, Func<HttpContext, Exception, Task>>
        {
            { typeof(NotFoundException), HandleNotFoundException },
            { typeof(UnauthorizedAccessException), HandleUnauthorizedAccessException },
            { typeof(ForbiddenAccessException), HandleForbiddenAccessException }
        };
    }

    public async ValueTask<bool> TryHandleAsync(HttpContext httpContext, Exception exception, CancellationToken cancellationToken)
    {
        var exceptionType = exception.GetType();

        if (!_exceptionHandlers.TryGetValue(exceptionType, out var handler))
        {
            await HandleUnknownException(httpContext, exception);

            return false;
        }

        await handler.Invoke(httpContext, exception);

        return true;
    }

    private async Task HandleNotFoundException(HttpContext httpContext, Exception exception)
    {
        var notFoundException = (NotFoundException)exception;

        httpContext.Response.StatusCode = StatusCodes.Status404NotFound;

        await httpContext.Response.WriteAsJsonAsync(
            new ProblemDetails
            {
                Status = StatusCodes.Status404NotFound,
                Type = "https://tools.ietf.org/html/rfc7231#section-6.5.4",
                Title = "The specified resource was not found.",
                Detail = notFoundException.Message
            });
    }

    private async Task HandleUnauthorizedAccessException(HttpContext httpContext, Exception exception)
    {
        httpContext.Response.StatusCode = StatusCodes.Status401Unauthorized;

        await httpContext.Response.WriteAsJsonAsync(
            new ProblemDetails
            {
                Status = StatusCodes.Status401Unauthorized,
                Title = "Unauthorized",
                Type = "https://tools.ietf.org/html/rfc7235#section-3.1",
                Detail = GetDetailsExceptionIfNotProduction(httpContext, exception)
            });
    }

    private async Task HandleForbiddenAccessException(HttpContext httpContext, Exception exception)
    {
        httpContext.Response.StatusCode = StatusCodes.Status403Forbidden;

        await httpContext.Response.WriteAsJsonAsync(
            new ProblemDetails
            {
                Status = StatusCodes.Status403Forbidden,
                Title = "Forbidden",
                Type = "https://tools.ietf.org/html/rfc7231#section-6.5.3",
                Detail = GetDetailsExceptionIfNotProduction(httpContext, exception)
            });
    }

    private async Task HandleUnknownException(HttpContext httpContext, Exception exception)
    {
        httpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;

        await httpContext.Response.WriteAsJsonAsync(
            new ProblemDetails
            {
                Status = StatusCodes.Status500InternalServerError,
                Title = "An error occurred while processing your request.",
                Type = "https://tools.ietf.org/html/rfc7231#section-6.6.1",
                Detail = GetDetailsExceptionIfNotProduction(httpContext, exception)
            });
    }

    private string GetDetailsExceptionIfNotProduction(HttpContext httpContext, Exception exception)
    {
        var hostEnvironment = httpContext
            .RequestServices
            .GetRequiredService<IWebHostEnvironment>();

        return !hostEnvironment.IsProduction() ? exception.Message : string.Empty;
    }
}
```

## Program

Editar `src/WebApi/Program.cs` y añadir.

```cs
app.UseExceptionHandler(_ => { });
```

## DependencyInjection

Editar `src/WebApi/DependencyInjection.cs` y añadir.

```cs
using CleanArchitecture.WebApi.Infrastructure;

services.AddExceptionHandler<CustomExceptionHandler>();
```
