---
sidebar_position: 1
title: "Pattern Result"
id: pattern-result
---

La idea es siempre devolver un `Result` en los **Commands** y **Queries**.

Para no tener que repetir siempre en los `IRequest<Result<TResponse>>`, generamos una interfaz que implementa `IRequest` y `IRequestHandler`.

## Result

Crear `src/Application/Common/Models/Result.cs`

```cs
namespace EmployeeControl.Domain.Common;

public class Result
{
    protected Result(bool succeeded, Dictionary<string, string[]> errors, int status = StatusCodes.Status200OK)
    {
        Succeeded = succeeded;
        Errors = errors;
        Status = status;
    }

    public bool Succeeded { get; private set; }

    public int Status { get; set; }

    public IDictionary<string, string[]> Errors { get; }

    public static Result Create(bool succeeded = true, int status = StatusCodes.Status200OK)
    {
        return new Result(succeeded, [], status);
    }

    public static Result<TValue> Create<TValue>(TValue? value, bool succeeded = true, int status = StatusCodes.Status200OK)
    {
        return new Result<TValue>(value, succeeded, [], status);
    }

    public static Result Success()
    {
        return new Result(true, []);
    }

    public static Result<TValue> Success<TValue>(TValue? value)
    {
        return new Result<TValue>(value, true, []);
    }

    public static Result Failure(Dictionary<string, string[]> errors)
    {
        return new Result(false, errors, StatusCodes.Status400BadRequest);
    }

    public static Result<TValue> Failure<TValue>(Dictionary<string, string[]> errors)
    {
        return new Result<TValue>(default, false, errors, StatusCodes.Status400BadRequest);
    }

    public static Result Failure(string code, string[] errors)
    {
        var result = Create(false, StatusCodes.Status400BadRequest);
        result.Errors.Add(code, errors);

        return result;
    }

    public static Result<TValue> Failure<TValue>(string code, string[] errors)
    {
        var result = Create<TValue>(default, false, StatusCodes.Status400BadRequest);
        result.Errors.Add(code, errors);

        return result;
    }

    public static Result Failure(string code, string error)
    {
        var result = Create(false, StatusCodes.Status400BadRequest);
        result.AddError(code, error);

        return result;
    }

    public static Result<TValue> Failure<TValue>(string code, string error)
    {
        var result = Create<TValue>(default, false, StatusCodes.Status400BadRequest);
        result.AddError(code, error);

        return result;
    }

    public static Result NotFound(string name, object key)
    {
        var result = Create(false, StatusCodes.Status404NotFound);
        result.AddError("NotFound", $"Entity \"{name}\" ({key}) was not found.");

        return result;
    }

    public static Result<TValue> NotFound<TValue>(string name, object key)
    {
        var result = Create<TValue>(default, false, StatusCodes.Status404NotFound);
        result.AddError("NotFound", $"Entity \"{name}\" ({key}) was not found.");

        return result;
    }

    public static Result Unauthorized()
    {
        var result = Create(false, StatusCodes.Status401Unauthorized);

        return result;
    }

    public static Result<TValue> Unauthorized<TValue>()
    {
        var result = Create<TValue>(default, false, StatusCodes.Status401Unauthorized);

        return result;
    }

    public static Result Forbidden()
    {
        var result = Create(false, StatusCodes.Status403Forbidden);

        return result;
    }

    public static Result<TValue> Forbidden<TValue>()
    {
        var result = Create<TValue>(default, false, StatusCodes.Status403Forbidden);

        return result;
    }

    public void AddError(string code, string error)
    {
        Succeeded = false;

        if (Errors.TryGetValue(code, out var value))
        {
            var newValue = value.Append(error).ToArray();
            Errors[code] = newValue;

            return;
        }

        Errors.Add(code, [error]);
    }
}

#pragma warning disable SA1402 // File may only contain a single type
public class Result<TValue> : Result
{
    protected internal Result(
        TValue? value,
        bool succeeded,
        Dictionary<string, string[]> errors,
        int status = StatusCodes.Status200OK)
        : base(succeeded, errors, status)
    {
        Value = value;
    }

    public TValue? Value { get; }

    public static implicit operator Result<TValue>(TValue? value)
    {
        return Create(value);
    }
}
#pragma warning disable SA1402 // File may only contain a single type
```

## ICommand

Crear `src/Application/Common/Interfaces/Messaging/ICommand.cs`

```cs
using CleanArchitecture.Application.Common.Models;
using MediatR;

namespace CleanArchitecture.Application.Common.Interfaces.Messaging;

public interface ICommand : IRequest<Result>
{
}

public interface ICommand<TResponse> : IRequest<Result<TResponse>>
{
}
```

## ICommandHandler

Crear `src/Application/Common/Interfaces/Messaging/ICommandHandler.cs`

```cs
using CleanArchitecture.Application.Common.Models;
using MediatR;

namespace CleanArchitecture.Application.Common.Interfaces.Messaging;

public interface ICommandHandler<TCommand> : IRequestHandler<TCommand, Result>
    where TCommand : ICommand
{
}

public interface ICommandHandler<TCommand, TResponse> : IRequestHandler<TCommand, Result<TResponse>>
    where TCommand : ICommand<TResponse>
{
}
```

## IQuery

Crear `src/Application/Common/Interfaces/Messaging/IQuery.cs`

```cs
using CleanArchitecture.Application.Common.Models;
using MediatR;

namespace CleanArchitecture.Application.Common.Interfaces.Messaging;

public interface IQuery<TResponse> : IRequest<Result<TResponse>>
{
}
```

## IQueryHandler

Crear `src/Application/Common/Interfaces/Messaging/IQueryHandler.cs`

```cs
using CleanArchitecture.Application.Common.Models;
using MediatR;

namespace CleanArchitecture.Application.Common.Interfaces.Messaging;

public interface IQueryHandler<TQuery, TResponse> : IRequestHandler<TQuery, Result<TResponse>>
    where TQuery : IQuery<TResponse>
{
}
```

## Modificar el SayHello

Modificar `src/Application/Hello/Queries/SayHelloQuery.cs`

```cs
using CleanArchitecture.Application.Common.Interfaces.Messaging;

namespace CleanArchitecture.Application.Hello.Queries;

public record SayHelloQuery(int Id) : IQuery<string>;
```

Modificar `src/Application/Hello/Queries/SayHelloHandler.cs`

```cs
using CleanArchitecture.Application.Common.Interfaces.Messaging;
using CleanArchitecture.Application.Common.Models;

namespace CleanArchitecture.Application.Hello.Queries;

internal sealed class SayHelloHandler : IQueryHandler<SayHelloQuery, string>
{
    public Task<Result<string>> Handle(SayHelloQuery request, CancellationToken cancellationToken)
    {
        var resultResponse = Result.Success("Hello .NET and MediatR");

        return Task.FromResult(resultResponse);
    }
}
```

Modificar `src/WebApi/Controllers/HelloController.cs`

```cs
using CleanArchitecture.Application.Common.Models;
using CleanArchitecture.Application.Hello.Queries;
using CleanArchitecture.WebApi.Infrastructure;
using Microsoft.AspNetCore.Mvc;

namespace CleanArchitecture.WebApi.Controllers;

[Route("api/hello")]
public class HelloController : ApiControllerBase
{
    [HttpGet]
    public async Task<ActionResult<Result<string>>> SayHello()
    {
        return await Sender.Send(new SayHelloQuery(15));
    }
}
```

Resultado con `new SayHelloQuery(5)`

```json
// 20240516233940
// https://localhost:7001/api/hello

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

Resultado con `new SayHelloQuery(15)`

```json
// 202105// 20240516234043
// https://localhost:7001/api/hello

{
  "value": "Hello .NET and MediatR",
  "succeeded": true,
  "status": 200,
  "errors": [

  ]
}
```
