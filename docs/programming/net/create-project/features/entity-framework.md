---
sidebar_position: 1
title: "EntityFramework"
id: entity-framework
---

## Dependencias Application

- Añadir `Microsoft.EntityFrameworkCore` en `src/Application/Application.csproj`

## Dependencias Infrastructure

- Añadir `Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore` en `src/Infrastructure/Infrastructure.csproj`
- Añadir `Microsoft.EntityFrameworkCore.Relational` em `src/Infrastructure/Infrastructure.csproj`
- Añadir `Microsoft.EntityFrameworkCore.Tools` en `src/Infrastructure/Infrastructure.csproj`
- Añadir `Npgsql.EntityFrameworkCore.PostgreSQL` en `src/Infrastructure/Infrastructure.csproj`

## Dependencias WebApi

- Añadir `Microsoft.EntityFrameworkCore.Design` en `src/WebApi/WebApi.csproj`

## Ef tools

`dotnet tool install -g dotnet-ef`

## Appsettings

Editar `src/WebApi/appsettings.Development.json` y añadir `ConnectionString`:

```xml
"ConnectionStrings": {
  "DefaultConnection": "User Id=postgres;Password=Password44!;Server=localhost;Port=5432;Database=CleanArchitecture;Pooling=true;"
}
```

## IAppDbContext

Crear `src/Application/Common/Interfaces/Persistence/IAppDbContext.cs`

```cs
using Microsoft.EntityFrameworkCore.Infrastructure;

namespace CleanArchitecture.Application.Common.Interfaces.Persistence;

public interface IAppDbContext
{
    // DbSet<Department> Departments { get; }

    DatabaseFacade Database { get; }

    Task<int> SaveChangesAsync(CancellationToken cancellationToken);
}
```

## AppDbContext

Crear `src/Infrastructure/Persistence/AppDbContext.cs`

```cs
using System.Reflection;
using CleanArchitecture.Application.Common.Interfaces.Persistence;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Infrastructure;

namespace CleanArchitecture.Infrastructure.Persistence;

public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options), IAppDbContext
{
    // public DbSet<Department> Departments => Set<Department>();
    public new DatabaseFacade Database => base.Database;

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());

        base.OnModelCreating(modelBuilder);
    }
}
```

## IEntityDomainEvent

Crear `src/Domain/Interfaces/IEntityDomainEvent.cs`

```cs
using CleanArchitecture.Domain.Common;

namespace CleanArchitecture.Domain.Interfaces;

public interface IEntityDomainEvent
{
    IReadOnlyCollection<BaseEvent> DomainEvents { get; }

    void AddDomainEvent(BaseEvent domainEvent);

    void RemoveDomainEvent(BaseEvent domainEvent);

    void ClearDomainEvents();
}
```

## BaseEntity

Crear `src/Domain/Common/BaseEntity.cs`.

```cs
using System.ComponentModel.DataAnnotations.Schema;
using CleanArchitecture.Domain.Interfaces;

namespace CleanArchitecture.Domain.Common;

public abstract class BaseEntity : IEntityDomainEvent
{
    private readonly List<BaseEvent> domainEvents;

    protected BaseEntity()
    {
        domainEvents = new List<BaseEvent>();
    }

    public string Id { get; private set; } = Guid.NewGuid().ToString();

    [NotMapped]
    public IReadOnlyCollection<BaseEvent> DomainEvents => domainEvents.AsReadOnly();

    public void AddDomainEvent(BaseEvent domainEvent)
    {
        domainEvents.Add(domainEvent);
    }

    public void RemoveDomainEvent(BaseEvent domainEvent)
    {
        domainEvents.Remove(domainEvent);
    }

    public void ClearDomainEvents()
    {
        domainEvents.Clear();
    }
}
```

## BaseAuditableEntity

Crear `src/Domain/Common/BaseAuditableEntity.cs`

```cs
namespace CleanArchitecture.Domain.Common;

public abstract class BaseAuditableEntity : BaseEntity
{
    public DateTimeOffset Created { get; set; }

    public string? CreatedBy { get; set; }

    public DateTimeOffset LastModified { get; set; }

    public string? LastModifiedBy { get; set; }
}
```

## Interceptors

Crear `src/Infrastructure/Persistence/Extensions/EntityEntryExtensions.cs`

```cs
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.ChangeTracking;

namespace CleanArchitecture.Infrastructure.Persistence.Extensions;

public static class EntityEntryExtensions
{
    public static bool HasChangedOwnedEntities(this EntityEntry entry)
    {
        return entry.References.Any(r =>
            r.TargetEntry != null &&
            r.TargetEntry.Metadata.IsOwned() &&
            r.TargetEntry.State is EntityState.Added or EntityState.Modified);
    }
}
```

Para "automatizar" los valores de `BaseAuditableEntity` al crear o actualizar datos en la base de datos.

Crear `src/Infrastructure/Persistence/Interceptors/DispatchDomainEventsInterceptor.cs`

```cs
using CleanArchitecture.Domain.Interfaces;
using MediatR;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Diagnostics;

namespace CleanArchitecture.Infrastructure.Persistence.Interceptors;

public class DispatchDomainEventsInterceptor(IMediator mediator) : SaveChangesInterceptor
{
    public override InterceptionResult<int> SavingChanges(DbContextEventData eventData, InterceptionResult<int> result)
    {
        DispatchDomainEvents(eventData.Context).GetAwaiter().GetResult();

        return base.SavingChanges(eventData, result);
    }

    public override async ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData eventData,
        InterceptionResult<int> result,
        CancellationToken cancellationToken = default)
    {
        await DispatchDomainEvents(eventData.Context);

        return await base.SavingChangesAsync(eventData, result, cancellationToken);
    }

    public async Task DispatchDomainEvents(DbContext? context)
    {
        if (context is null)
        {
            return;
        }

        var entities = context.ChangeTracker
            .Entries<IEntityDomainEvent>()
            .Where(e => e.Entity.DomainEvents.Count != 0)
            .Select(e => e.Entity);

        var baseEntities = entities as IEntityDomainEvent[] ?? entities.ToArray();

        var domainEvents = baseEntities
            .SelectMany(e => e.DomainEvents)
            .ToList();

        baseEntities.ToList().ForEach(e => e.ClearDomainEvents());

        foreach (var domainEvent in domainEvents)
        {
            await mediator.Publish(domainEvent);
        }
    }
}
```

Crear `src/Infrastructure/Persistence/Interceptors/AuditableEntityInterceptor.cs`

```cs
using CleanArchitecture.Application.Common.Interfaces.Common;
using CleanArchitecture.Application.Common.Interfaces.Users;
using CleanArchitecture.Domain.Common;
using CleanArchitecture.Infrastructure.Persistence.Extensions;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Diagnostics;

namespace CleanArchitecture.Infrastructure.Persistence.Interceptors;

public class AuditableEntityInterceptor(ICurrentUserService currentUserService, IDateTimeProvider dateTimeProvider)
    : SaveChangesInterceptor
{
    public override InterceptionResult<int> SavingChanges(DbContextEventData eventData, InterceptionResult<int> result)
    {
        UpdateEntities(eventData.Context);

        return base.SavingChanges(eventData, result);
    }

    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData eventData,
        InterceptionResult<int> result,
        CancellationToken cancellationToken = default)
    {
        UpdateEntities(eventData.Context);

        return base.SavingChangesAsync(eventData, result, cancellationToken);
    }

    private void UpdateEntities(DbContext? context)
    {
        if (context is null)
        {
            return;
        }

        foreach (var entry in context.ChangeTracker.Entries<BaseAuditableEntity>())
        {
            if (entry.State is EntityState.Added)
            {
                entry.Entity.CreatedBy = currentUserService.Id;
                entry.Entity.Created = dateTimeProvider.UtcNow;
            }

            if (entry.State != EntityState.Added && entry.State != EntityState.Modified && !entry.HasChangedOwnedEntities())
            {
                continue;
            }

            entry.Entity.LastModifiedBy = currentUserService.Id;
            entry.Entity.LastModified = dateTimeProvider.UtcNow;
        }
    }
}

```

## Manage Seeds

Crear `src/Infrastructure/Persistence/Seeds/AppDbContextInitialize.cs`

```cs
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Logging;

namespace CleanArchitecture.Infrastructure.Persistence.Seeds;

public class AppDbContextInitialize(AppDbContext context, ILogger<AppDbContextInitialize> logger)
{
    public async Task InitialiseAsync()
    {
        try
        {
            await context.Database.MigrateAsync();
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "An error occurred while initialising the database.");
            throw;
        }
    }

    public async Task SeedAsync()
    {
        try
        {
            await TrySeedAsync();
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "An error occurred while seeding the database.");
            throw;
        }
    }

    private Task TrySeedAsync()
    {
        // await CreateRolesAsync();
        // await CreateUsersAsync();

        return Task.CompletedTask;
    }
}
```

Crear `src/Infrastructure/Persistence/Seeds/InitialiseExtensions.cs`

```cs
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;

namespace CleanArchitecture.Infrastructure.Persistence.Seeds;

public static class InitialiseExtensions
{
    public static async Task InitialiseDatabaseAsync(this WebApplication app)
    {
        using var scope = app.Services.CreateScope();

        var initialise = scope.ServiceProvider.GetRequiredService<AppDbContextInitialize>();

        await initialise.InitialiseAsync();

        await initialise.SeedAsync();
    }
}
```

## Configuración

Editar `src/WebApi/Program.cs` para pasar los parámetros de `src/Infrastructure/DependencyInjection.AddInfrastructure`.

```cs
builder.Services
    .AddApplication()
    .AddInfrastructure(builder.Configuration, builder.Environment)
    .AddWebApi();
```

Editar `src/Infrastructure/DependencyInjection.cs`

Añadir parámetros `IConfiguration configuration` y `IWebHostEnvironment environment` en `AddInfrastructure`

```cs
public static IServiceCollection AddInfrastructureServices(
        this IServiceCollection services,
        IConfiguration configuration,
        IWebHostEnvironment environment)
{
    // ...
}
```

Añadir configuración del servicio

```cs
// Database.
// Interceptors EntityFramework.
services.AddScoped<ISaveChangesInterceptor, AuditableEntityInterceptor>();
services.AddScoped<ISaveChangesInterceptor, DispatchDomainEventsInterceptor>();

// Db service.
services.AddDbContext<AppDbContext>(
    (provider, options) =>
    {
        options.AddInterceptors(provider.GetServices<ISaveChangesInterceptor>());
        options.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
        options.UseNpgsql(configuration.GetConnectionString("DefaultConnection"));

        if (!environment.IsProduction())
        {
            options.EnableSensitiveDataLogging();
        }
    });

services.AddScoped<IAppDbContext>(provider => provider.GetRequiredService<AppDbContext>());
services.AddScoped<AppDbContextInitialize>();
```

Editar `src/WebApi/DependencyInjection.cs` y añadir:

```cs
services.AddDatabaseDeveloperPageExceptionFilter();
```

## Migraciones

```bash
cd src/WebApi

dotnet ef migrations add Initial -p ../Infrastructure/Infrastructure.csproj  -c AppDbContext  -o ../Infrastructure/Persistence/Migrations

dotnet ef database update -c AppDbContext
```

```bash
Build started...
Build succeeded.
[22:12:46 WRN] Sensitive data logging is enabled. Log entries and exception messages may include sensitive application data; this mode should only be enabled during development.
[22:12:46 INF] Executed DbCommand (60ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE DATABASE "CleanArchitecture";
[22:12:46 INF] Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "__EFMigrationsHistory" (
    "MigrationId" character varying(150) NOT NULL,
    "ProductVersion" character varying(32) NOT NULL,
    CONSTRAINT "PK___EFMigrationsHistory" PRIMARY KEY ("MigrationId")
);
[22:12:46 INF] Executed DbCommand (19ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT EXISTS (
    SELECT 1 FROM pg_catalog.pg_class c
    JOIN pg_catalog.pg_namespace n ON n.oid=c.relnamespace
    WHERE n.nspname='public' AND
          c.relname='__EFMigrationsHistory'
)
[22:12:46 INF] Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT "MigrationId", "ProductVersion"
FROM "__EFMigrationsHistory"
ORDER BY "MigrationId";
[22:12:46 INF] Applying migration '20240516200938_Initial'.
[22:12:46 INF] Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
INSERT INTO "__EFMigrationsHistory" ("MigrationId", "ProductVersion")
VALUES ('20240516200938_Initial', '8.0.5');
Done.
```
