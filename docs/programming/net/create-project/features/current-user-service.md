---
sidebar_position: 1
title: "CurrentUserService"
id: current-user-service
---

Servicio para obtener datos del usuario actual.

Crear `src/Application/Common/Interfaces/Users/ICurrentUserService.cs`

```cs
namespace CleanArchitecture.Application.Common.Interfaces.Users;

public interface ICurrentUserService
{
    Guid Id { get; }

    IEnumerable<Guid> Roles { get; }
}
```

Crear `src/WebApi/Services/CurrentUserService.cs`

```cs
using System.Security.Claims;
using CleanArchitecture.Application.Common.Interfaces.Users;

namespace CleanArchitecture.WebApi.Services;

public class CurrentUserService(IHttpContextAccessor httpContextAccessor)
    : ICurrentUserService
{
    public Guid Id => Guid.Parse(httpContextAccessor.HttpContext?.User.FindFirstValue(ClaimTypes.Sid) ?? string.Empty);

    public IEnumerable<Guid> Roles =>
        httpContextAccessor
            .HttpContext?
            .User.FindAll(ClaimTypes.Role)
            .Select(r => Guid.Parse(r.Value)) ?? [];
}
```

Editar `src/WebApi/DependencyInjection.cs`

```cs
services.AddHttpContextAccessor();

services.AddScoped<ICurrentUserService, CurrentUserService>();
```
