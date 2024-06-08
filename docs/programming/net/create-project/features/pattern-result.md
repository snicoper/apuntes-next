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
    protected Result(bool succeeded, Dictionary<string, string[]> errors)
    {
        Succeeded = succeeded;
        Errors = errors;
    }

    public bool Succeeded { get; private set; }

    public int Status { get; set; }

    public IDictionary<string, string[]> Errors { get; }

    public static Result Create()
    {
        return new Result(true, []);
    }

    public static Result<TValue> Create<TValue>(TValue? value)
    {
        return new Result<TValue>(value, true, []);
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
        return new Result(false, errors);
    }

    public static Result<TValue> Failure<TValue>(Dictionary<string, string[]> errors)
    {
        return new Result<TValue>(default, false, errors);
    }

    public static Result Failure(string code, string[] errors)
    {
        var result = Create();
        result.Errors.Add(code, errors);
        result.Succeeded = false;

        return result;
    }

    public static Result<TValue> Failure<TValue>(string code, string[] errors)
    {
        var result = Create<TValue>(default);
        result.Errors.Add(code, errors);
        result.Succeeded = false;

        return result;
    }

    public static Result Failure(string code, string error)
    {
        var result = Create();
        result.AddError(code, error);
        result.Succeeded = false;

        return result;
    }

    public static Result<TValue> Failure<TValue>(string code, string error)
    {
        var result = Create<TValue>(default);
        result.AddError(code, error);
        result.Succeeded = false;

        return result;
    }

    public void AddError(string code, string error)
    {
        Succeeded = false;
        Errors.Add(code, [error]);
    }
}

#pragma warning disable SA1402 // File may only contain a single type
public class Result<TValue> : Result
{
    protected internal Result(TValue? value, bool succeeded, Dictionary<string, string[]> errors)
        : base(succeeded, errors)
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
