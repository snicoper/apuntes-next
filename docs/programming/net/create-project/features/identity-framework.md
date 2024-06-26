---
sidebar_position: 1
title: "IdentityFramework"
id: identity-framework
---

Configuración básica de IdentityFramework.

## Añadir paquete

- Añadir `Microsoft.AspNetCore.Identity.UI` en el proyecto `src/Domain/Domain.csproj`.
- Añadir `Microsoft.AspNetCore.Identity.EntityFrameworkCore` en el proyecto `src/Infrastructure/Infrastructure.csproj`
- Añadir `Microsoft.AspNetCore.Authentication.JwtBearer` en el proyecto `src/Application/Application.csproj`

## JwtSettings

Editar `src/WebApi/appsettings.Development.json`

```json
"Jwt": {
  "AccessTokenLifeTimeMinutes": 10,
  "RefreshTokenLifeTimeDays": 30,
  "Issuer": "https://localhost:7000",
  "Audience": "https://localhost:7000",
  "Key": "S3cr3t_K3y!.123_S3cr3t_K3y!.123-Thh!js"
}
```

Cambiar el puerto según `src/WebApi/Properties/launchSettings.json`

Crear `src/Application/Common/Models/Settings/JwtSettings.cs` para tipar las opciones de configuración.

```cs
using System.ComponentModel.DataAnnotations;

namespace CleanArchitecture.Application.Common.Models.Settings;

public class JwtSettings
{
    public const string SectionName = "Jwt";

    public const string JwtAccessTokenLifeTimeMinutes = "Jwt:AccessTokenLifeTimeMinutes";
    public const string JwtRefreshTokenLifeTimeDays = "Jwt:RefreshTokenLifeTimeDays";
    public const string JwtIssuer = "Jwt:Issuer";
    public const string JwtAudience = "Jwt:Audience";
    public const string JwtKey = "Jwt:Key";

    [Range(10, int.MaxValue)]
    public int AccessTokenLifeTimeMinutes { get; set; }

    [Range(1, int.MaxValue)]
    public int RefreshTokenLifeTimeDays { get; set; }

    [Required]
    public string? Issuer { get; set; }

    [Required]
    public string? Audience { get; set; }

    [MinLength(32)]
    public string? Key { get; set; }
}
```

Editar `src/Application/DependencyInjection.cs` y añadir

```cs
// Strongly typed options validations.
services.AddOptions<JwtSettings>()
    .Bind(configuration.GetSection(JwtSettings.SectionName))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

## CustomClaimsPrincipalFactory

Crear `src/Application/Common/Constants/CustomClaims.cs` para añadir claims personalizadas.

```cs
namespace CleanArchitecture.Application.Common.Constants;

public static class CustomClaims
{
    // public const string CompanyId = "companyId";
}
```

Crear `src/Application/Common/Services/CustomClaimsPrincipalFactory.cs`

```cs
using System.Security.Claims;
using CleanArchitecture.Domain.Entities;
using Microsoft.AspNetCore.Identity;
using Microsoft.Extensions.Options;

namespace CleanArchitecture.Application.Common.Services;

public class CustomClaimsPrincipalFactory(UserManager<User> userManager, IOptions<IdentityOptions> optionsAccessor)
    : UserClaimsPrincipalFactory<User>(userManager, optionsAccessor)
{
    protected override async Task<ClaimsIdentity> GenerateClaimsAsync(User user)
    {
        var identity = await base.GenerateClaimsAsync(user);

        // identity.AddClaim(new Claim(CustomClaims.CompanyId, user.CompanyId));

        return identity;
    }
}
```

## User

Crear `src/Domain/Entities/User.cs`

```cs
using System.ComponentModel.DataAnnotations.Schema;
using CleanArchitecture.Domain.Common;
using CleanArchitecture.Domain.Interfaces;
using Microsoft.AspNetCore.Identity;

namespace CleanArchitecture.Domain.Entities;

public class User : IdentityUser, IEntityDomainEvent
{
    private readonly List<BaseEvent> _domainEvents = new();

    public string? FirstName { get; set; }

    public string? LastName { get; set; }

    public string? RefreshToken { get; set; }

    public bool Active { get; set; }

    public DateTimeOffset? RefreshTokenExpiryTime { get; set; }

    public DateTimeOffset EntryDate { get; set; }

    [NotMapped]
    public IReadOnlyCollection<BaseEvent> DomainEvents => _domainEvents.AsReadOnly();

    public void AddDomainEvent(BaseEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }

    public void RemoveDomainEvent(BaseEvent domainEvent)
    {
        _domainEvents.Remove(domainEvent);
    }

    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
}
```

## UserConfiguration

Crear `src/Infrastructure/Persistence/Configurations/UserConfiguration.cs`

```cs
using CleanArchitecture.Domain.Entities;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace CleanArchitecture.Infrastructure.Configurations;

internal class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        // Indexes.
        builder.HasIndex(au => au.Email)
            .IsUnique();

        builder.HasIndex(au => au.RefreshToken)
            .IsUnique();

        // Properties.
        builder.Property(au => au.Email)
            .IsRequired()
            .HasMaxLength(256);

        builder.Property(au => au.FirstName)
            .HasMaxLength(256)
            .IsRequired();

        builder.Property(au => au.LastName)
            .HasMaxLength(256)
            .IsRequired();

        builder.Property(au => au.RefreshToken)
            .HasMaxLength(256);

        builder.Property(au => au.Active);

        builder.Property(au => au.RefreshTokenExpiryTime);

        builder.Property(au => au.EntryDate)
            .IsRequired();
    }
}
```

## AppDbContext

Editar `src/Infrastructure/Persistence/AppDbContext.cs` y remplazar por:

```cs
public class AppDbContext(DbContextOptions<AppDbContext> options)
    : IdentityDbContext<User, IdentityRole<Guid>, Guid>(options), IAppDbContext
{
    // public DbSet<Department> Departments => Set<Department>();

    public new DatabaseFacade Database => base.Database;

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());

        base.OnModelCreating(builder);
    }
}
```

## DependencyInjection

Editar `src/Infrastructure/DependencyInjection.cs`.

```cs
// Identity.
services.AddAuthorizationBuilder();

services
    .AddIdentityCore<User>()
    .AddRoles<IdentityRole<Guid>>()
    .AddEntityFrameworkStores<AppDbContext>()
    .AddClaimsPrincipalFactory<CustomClaimsPrincipalFactory>()
    .AddDefaultTokenProviders();

services.Configure<IdentityOptions>(
    options =>
    {
        // Default Password settings.
        options.Password.RequireDigit = false;
        options.Password.RequireLowercase = false;
        options.Password.RequireNonAlphanumeric = false;
        options.Password.RequireUppercase = false;
        options.Password.RequiredLength = 6;
        options.Password.RequiredUniqueChars = 0;

        // Default User settings.
        options.User.RequireUniqueEmail = true;

        // Default SignIn settings.
        options.SignIn.RequireConfirmedAccount = true;
    });

services.AddAuthentication(
        options =>
        {
            options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
        })
    .AddJwtBearer(
        options =>
        {
            var jwtKey = configuration[JwtSettings.JwtKey] ?? string.Empty;
            var jwtIssuer = configuration[JwtSettings.JwtIssuer];
            var jwtAudience = configuration[JwtSettings.JwtAudience];

            options.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidateLifetime = true,
                ValidateIssuerSigningKey = true,
                ValidIssuer = jwtIssuer,
                ValidAudience = jwtAudience,
                IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtKey)),
                ClockSkew = TimeSpan.Zero
            };
        });
```

Editar `src/WebApi/Program.cs` y añadir.

```cs
// Initialise Database in development.
if (app.Environment.IsDevelopment())
{
    await app.InitialiseDatabaseAsync();
}
else
{
    app.UseHsts();
}

app.UseAuthentication();
app.UseAuthorization();
```

## Authorize

Editar `src/WebApi/Infrastructure/ApiControllerBase.cs` y añadir el decorador [Authorize]

```cs
[Authorize]
[ApiController]
[Produces("application/json")]
[ProducesResponseType(StatusCodes.Status401Unauthorized)]
public class ApiControllerBase : ControllerBase
{
    private ISender? _sender;

    protected ISender Sender => _sender ??= HttpContext.RequestServices.GetRequiredService<ISender>();
}
```

Editar `src/WebApi/Controllers/HelloController.cs` y en el método `SayHello` añadir el decorador [AllowAnonymous].

```cs
[HttpGet]
[AllowAnonymous]
```

## Migraciones

Si existe, eliminar el directorio `src/Infrastructure/Persistence/Migrations` que se creo al añadir [EntityFramework](./entity-framework.md)

```bash
cd src/WebApi

dotnet ef migrations add Initial -p ../Infrastructure/Infrastructure.csproj  -c AppDbContext  -o ../Infrastructure/Persistence/Migrations

dotnet ef database update -c AppDbContext
```

:::danger
**StyleCop** genera un error si se añade un `using` que no se utiliza, por lo que se ha de comprobar los archivos
generados en `src/Infrastructure/Persistence/Migrations` y eliminar los `using` que no se usen.
:::

```bash
➜ dotnet ef database update -c AppDbContext
Build started...
Build succeeded.
[01:47:50 WRN] Sensitive data logging is enabled. Log entries and exception messages may include sensitive application data; this mode should only be enabled during development.
[01:47:50 INF] Executed DbCommand (29ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT EXISTS (
    SELECT 1 FROM pg_catalog.pg_class c
    JOIN pg_catalog.pg_namespace n ON n.oid=c.relnamespace
    WHERE n.nspname='public' AND
          c.relname='__EFMigrationsHistory'
)
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT "MigrationId", "ProductVersion"
FROM "__EFMigrationsHistory"
ORDER BY "MigrationId";
[01:47:50 INF] Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT EXISTS (
    SELECT 1 FROM pg_catalog.pg_class c
    JOIN pg_catalog.pg_namespace n ON n.oid=c.relnamespace
    WHERE n.nspname='public' AND
          c.relname='__EFMigrationsHistory'
)
[01:47:50 INF] Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT EXISTS (
    SELECT 1 FROM pg_catalog.pg_class c
    JOIN pg_catalog.pg_namespace n ON n.oid=c.relnamespace
    WHERE n.nspname='public' AND
          c.relname='__EFMigrationsHistory'
)
[01:47:50 INF] Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT "MigrationId", "ProductVersion"
FROM "__EFMigrationsHistory"
ORDER BY "MigrationId";
[01:47:50 INF] Applying migration '20240609234731_Initial'.
[01:47:50 INF] Executed DbCommand (6ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetRoles" (
    "Id" uuid NOT NULL,
    "Name" character varying(256),
    "NormalizedName" character varying(256),
    "ConcurrencyStamp" text,
    CONSTRAINT "PK_AspNetRoles" PRIMARY KEY ("Id")
);
[01:47:50 INF] Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUsers" (
    "Id" uuid NOT NULL,
    "FirstName" character varying(256) NOT NULL,
    "LastName" character varying(256) NOT NULL,
    "RefreshToken" character varying(256),
    "Active" boolean NOT NULL,
    "RefreshTokenExpiryTime" timestamp with time zone,
    "EntryDate" timestamp with time zone NOT NULL,
    "UserName" character varying(256),
    "NormalizedUserName" character varying(256),
    "Email" character varying(256) NOT NULL,
    "NormalizedEmail" character varying(256),
    "EmailConfirmed" boolean NOT NULL,
    "PasswordHash" text,
    "SecurityStamp" text,
    "ConcurrencyStamp" text,
    "PhoneNumber" text,
    "PhoneNumberConfirmed" boolean NOT NULL,
    "TwoFactorEnabled" boolean NOT NULL,
    "LockoutEnd" timestamp with time zone,
    "LockoutEnabled" boolean NOT NULL,
    "AccessFailedCount" integer NOT NULL,
    CONSTRAINT "PK_AspNetUsers" PRIMARY KEY ("Id")
);
[01:47:50 INF] Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetRoleClaims" (
    "Id" integer GENERATED BY DEFAULT AS IDENTITY,
    "RoleId" uuid NOT NULL,
    "ClaimType" text,
    "ClaimValue" text,
    CONSTRAINT "PK_AspNetRoleClaims" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_AspNetRoleClaims_AspNetRoles_RoleId" FOREIGN KEY ("RoleId") REFERENCES "AspNetRoles" ("Id") ON DELETE CASCADE
);
[01:47:50 INF] Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUserClaims" (
    "Id" integer GENERATED BY DEFAULT AS IDENTITY,
    "UserId" uuid NOT NULL,
    "ClaimType" text,
    "ClaimValue" text,
    CONSTRAINT "PK_AspNetUserClaims" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_AspNetUserClaims_AspNetUsers_UserId" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id") ON DELETE CASCADE
);
[01:47:50 INF] Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUserLogins" (
    "LoginProvider" text NOT NULL,
    "ProviderKey" text NOT NULL,
    "ProviderDisplayName" text,
    "UserId" uuid NOT NULL,
    CONSTRAINT "PK_AspNetUserLogins" PRIMARY KEY ("LoginProvider", "ProviderKey"),
    CONSTRAINT "FK_AspNetUserLogins_AspNetUsers_UserId" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id") ON DELETE CASCADE
);
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUserRoles" (
    "UserId" uuid NOT NULL,
    "RoleId" uuid NOT NULL,
    CONSTRAINT "PK_AspNetUserRoles" PRIMARY KEY ("UserId", "RoleId"),
    CONSTRAINT "FK_AspNetUserRoles_AspNetRoles_RoleId" FOREIGN KEY ("RoleId") REFERENCES "AspNetRoles" ("Id") ON DELETE CASCADE,
    CONSTRAINT "FK_AspNetUserRoles_AspNetUsers_UserId" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id") ON DELETE CASCADE
);
[01:47:50 INF] Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUserTokens" (
    "UserId" uuid NOT NULL,
    "LoginProvider" text NOT NULL,
    "Name" text NOT NULL,
    "Value" text,
    CONSTRAINT "PK_AspNetUserTokens" PRIMARY KEY ("UserId", "LoginProvider", "Name"),
    CONSTRAINT "FK_AspNetUserTokens_AspNetUsers_UserId" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id") ON DELETE CASCADE
);
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "IX_AspNetRoleClaims_RoleId" ON "AspNetRoleClaims" ("RoleId");
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE UNIQUE INDEX "RoleNameIndex" ON "AspNetRoles" ("NormalizedName");
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "IX_AspNetUserClaims_UserId" ON "AspNetUserClaims" ("UserId");
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "IX_AspNetUserLogins_UserId" ON "AspNetUserLogins" ("UserId");
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "IX_AspNetUserRoles_RoleId" ON "AspNetUserRoles" ("RoleId");
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "EmailIndex" ON "AspNetUsers" ("NormalizedEmail");
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE UNIQUE INDEX "IX_AspNetUsers_Email" ON "AspNetUsers" ("Email");
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE UNIQUE INDEX "IX_AspNetUsers_RefreshToken" ON "AspNetUsers" ("RefreshToken");
[01:47:50 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE UNIQUE INDEX "UserNameIndex" ON "AspNetUsers" ("NormalizedUserName");
[01:47:50 INF] Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
INSERT INTO "__EFMigrationsHistory" ("MigrationId", "ProductVersion")
VALUES ('20240609234731_Initial', '8.0.6');
Done.
```
