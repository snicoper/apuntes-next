---
sidebar_position: 1
title: "IDateTimeProvider"
id: i-date-time-service
---

Crear `src/Application/Common/Interfaces/IDateTimeProvider.cs`

```cs
namespace CleanArchitecture.Application.Common.Interfaces;

public interface IDateTimeProvider
{
    TimeProvider TimeProvider { get; }

    DateTimeOffset UtcNow { get; }
}
```

Crear `src/Infrastructure/Services/Common/DateTimeProvider.cs`

```cs
using CleanArchitecture.Application.Common.Interfaces;

namespace CleanArchitecture.Infrastructure.Services;

public class DateTimeProvider(TimeProvider timeProvider) : IDateTimeProvider
{
    public TimeProvider TimeProvider { get; } = timeProvider;

    public DateTimeOffset UtcNow => TimeProvider.GetUtcNow();
}
```

Editar `src/Infrastructure/DependencyInjection.cs` y añadir.

```cs
services.AddSingleton(TimeProvider.System);
services.AddScoped<IDateTimeProvider, DateTimeProvider>();
```
