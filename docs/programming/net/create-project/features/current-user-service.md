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
    string Id { get; }

    IEnumerable<string> Roles { get; }
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
    public string Id => httpContextAccessor.HttpContext?.User.FindFirstValue(ClaimTypes.Sid) ?? string.Empty;

    public IEnumerable<string> Roles =>
        httpContextAccessor.HttpContext?.User.FindAll(ClaimTypes.Role).Select(r => r.Value) ?? new List<string>();
}
```

Editar `src/Infrastructure/DependencyInjection.cs`

```cs
services.AddSingleton(TimeProvider.System);
```

Editar `src/WebApi/DependencyInjection.cs`

```cs
services.AddHttpContextAccessor();

services.AddScoped<ICurrentUserService, CurrentUserService>();
```
