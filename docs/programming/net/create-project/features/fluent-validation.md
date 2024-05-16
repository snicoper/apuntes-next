---
sidebar_position: 1
title: "FluentValidation"
id: fluentvalidation
---

Ver [CustomExceptionHandler](./custom-exception-handler.md)

Añadir el paquete `FluentValidation.DependencyInjectionExtensions` en `src/Application/Application.csproj`

## DependencyInjection

```csharp
using FluentValidation;

...

// FluentValidator.
services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());
```

## Middleware de MediatR

### CustomValidationException

Crear excepción personalizada en `src/Application/Common/Exceptions/CustomValidationException.cs`

```cs
using CleanArchitecture.Application.Common.Extensions;
using FluentValidation.Results;

namespace CleanArchitecture.Application.Common.Exceptions;

public class CustomValidationException() : Exception("One or more validation failures have occurred.")
{
    public CustomValidationException(IReadOnlyCollection<ValidationFailure> failures)
        : this()
    {
        Errors = failures
            .GroupBy(e => e.PropertyName, e => e.ErrorMessage)
            .ToDictionary(failureGroup => failureGroup.Key.LowerCaseFirst(), failureGroup => failureGroup.ToArray());
    }

    public IDictionary<string, string[]> Errors { get; } = new Dictionary<string, string[]>();
}
```

Esto agrupara todos los errores de validación por el nombre de la propiedad.

### StringExtensions

Para devolver los nombres de propiedades con **camelCase** en vez de **CamelCase**, creamos un método de extensión en
`src/Application/Common/Extensions/StringExtensions.cs`

```cs
namespace CleanArchitecture.Application.Common.Extensions;

public static class StringExtensions
{
    /// <summary>
    /// Devolver un string con la primera letra en minúscula.
    /// </summary>
    /// <param name="value">String a convertir.</param>
    /// <returns>El string con la primera letra en minúsculas.</returns>
    public static string LowerCaseFirst(this string value)
    {
        var result = string.IsNullOrEmpty(value) ? value : $"{value[..1].ToLower()}{value[1..]}";

        return result;
    }

    /// <summary>
    /// Devolver un string con la primera letra en mayúscula.
    /// </summary>
    /// <param name="value">String a convertir.</param>
    /// <returns>El string con la primera letra en mayúsculas.</returns>
    public static string UpperCaseFirst(this string value)
    {
        var result = string.IsNullOrEmpty(value) ? value : $"{value[..1].ToUpper()}{value[1..]}";

        return result;
    }
}
```

### ValidationBehaviour

Crear middleware `src/Application/Common/Behaviours/ValidationBehaviour.cs`

```cs
using CleanArchitecture.Application.Common.Exceptions;
using FluentValidation;
using MediatR;

namespace CleanArchitecture.Application.Common.Behaviours;

public class ValidationBehaviour<TRequest, TResponse>(IEnumerable<IValidator<TRequest>> validators)
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : notnull
{
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        if (!validators.Any())
        {
            return await next();
        }

        var context = new ValidationContext<TRequest>(request);

        var validationResults = await Task.WhenAll(validators.Select(v => v.ValidateAsync(context, cancellationToken)));

        var failures = validationResults
            .Where(r => r.Errors.Count != 0)
            .SelectMany(r => r.Errors)
            .ToList();

        if (failures.Count != 0)
        {
            throw new CustomValidationException(failures);
        }

        return await next();
    }
}
```

Este middleware comprobará si tiene algún error de validación y lanzará una excepción personalizada con los errores.

## Añadir middleware a MediatR

Editar `src/Application/DependencyInjection.cs` y añadir el `ValidationBehaviour`.

```cs
config.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly());
config.AddBehavior(typeof(IPipelineBehavior<,>), typeof(UnhandledExceptionBehaviour<,>));
config.AddBehavior(typeof(IPipelineBehavior<,>), typeof(ValidationBehaviour<,>));
```

## CustomExceptionHandler

Editar `src/WebApi/Infrastructure/CustomExceptionHandler.cs` para manejar errores de `CustomValidationException`.

En el constructor añadir el **handler** en `_exceptionHandlers`

```cs
// Register known exception types and handlers.
_exceptionHandlers = new Dictionary<Type, Func<HttpContext, Exception, Task>>
{
    { typeof(CustomValidationException), HandleValidationException },
    { typeof(NotFoundException), HandleNotFoundException },
    { typeof(UnauthorizedAccessException), HandleUnauthorizedAccessException },
    { typeof(ForbiddenAccessException), HandleForbiddenAccessException }
};
```

Crear método HandleValidationException en la clase `CustomExceptionHandler`.

```cs
private async Task HandleValidationException(HttpContext httpContext, Exception exception)
{
    var customValidationException = (CustomValidationException)exception;

    httpContext.Response.StatusCode = StatusCodes.Status400BadRequest;

    await httpContext.Response.WriteAsJsonAsync(
        new ValidationProblemDetails(customValidationException.Errors)
        {
            Status = StatusCodes.Status400BadRequest, Type = "https://tools.ietf.org/html/rfc7231#section-6.5.1"
        });
}
```

## Probar que todo esta correcto

El ejemplo es un poco tonto pero para probar ya vale.

Editar `src/Application/Hello/Queries/SayHelloQuery.cs` para añadir un parametro `int Id`.

```cs
using MediatR;

namespace CleanArchitecture.Application.Hello.Queries;

public record SayHelloQuery(int Id) : IRequest<string>;
```

Crear clase de validación ``src/Application/Hello/Queries/SayHelloValidator.cs`

```cs
using FluentValidation;

namespace CleanArchitecture.Application.Hello.Queries;

public class SayHelloValidator : AbstractValidator<SayHelloQuery>
{
    public SayHelloValidator()
    {
        RuleFor(r => r.Id)
            .NotEmpty()
            .GreaterThan(10);
    }
}
```

Editar `src/WebApi/Controllers/HelloController.cs` y en el método `SayHello`, cambiar por:

```cs
return await Sender.Send(new SayHelloQuery(5));
```

```json
// 20240516033408
// https://localhost:7236/api/hello

{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "id": [
      "'Id' must be greater than '10'."
    ]
  }
}
```
