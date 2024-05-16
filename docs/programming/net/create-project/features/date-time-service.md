---
sidebar_position: 1
title: "IDateTimeService"
id: i-date-time-service
---

Crear `src/Application/Common/Interfaces/Common/IDateTimeService.cs`

```cs
namespace CleanArchitecture.Application.Common.Interfaces.Common;

public interface IDateTimeService
{
    /// <summary>
    /// Obtener el <see cref="TimeProvider" />.
    /// </summary>
    TimeProvider TimeProvider { get; }

    /// <summary>
    /// Alias de TimeProvider.GetUtcNow().
    /// </summary>
    DateTimeOffset UtcNow { get; }
}
```

Crear `src/Infrastructure/Services/Common/DateTimeService.cs`

```cs
using CleanArchitecture.Application.Common.Interfaces.Common;

namespace CleanArchitecture.Infrastructure.Services;

public class DateTimeService(TimeProvider timeProvider) : IDateTimeService
{
    public TimeProvider TimeProvider { get; } = timeProvider;

    public DateTimeOffset UtcNow => TimeProvider.GetUtcNow();
}
```

Editar `src/Infrastructure/DependencyInjection.cs` y añadir.

```cs
services.AddSingleton(TimeProvider.System);
services.AddScoped<IDateTimeService, DateTimeService>();
```
