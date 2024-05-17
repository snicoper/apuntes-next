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

## Configuración

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

Crear `src/Application/Common/Models/Settings/Settings/JwtSettings.cs` para tipar las opciones de configuración.

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

Editar `src/Application/Common/Models/Settings/JwtSettings.cs` y añadir

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

public class CustomClaimsPrincipalFactory(UserManager<ApplicationUser> userManager, IOptions<IdentityOptions> optionsAccessor)
    : UserClaimsPrincipalFactory<ApplicationUser>(userManager, optionsAccessor)
{
    protected override async Task<ClaimsIdentity> GenerateClaimsAsync(ApplicationUser user)
    {
        var identity = await base.GenerateClaimsAsync(user);

        // identity.AddClaim(new Claim(CustomClaims.CompanyId, user.CompanyId));

        return identity;
    }
}
```

## Entidad ApplicationUser

Crear `src/Domain/Entities/ApplicationUser.cs`

```cs
using System.ComponentModel.DataAnnotations.Schema;
using CleanArchitecture.Domain.Common;
using CleanArchitecture.Domain.Interfaces;
using Microsoft.AspNetCore.Identity;

namespace CleanArchitecture.Domain.Entities;

public class ApplicationUser : IdentityUser, IEntityDomainEvent
{
    private readonly List<BaseEvent> _domainEvents = new();

    public string? FirstName { get; set; }

    public string? LastName { get; set; }

    public string? RefreshToken { get; set; }

    public bool Active { get; set; }

    public DateTimeOffset? RefreshTokenExpiryTime { get; set; }

    public DateTimeOffset EntryDate { get; set; }

    public ICollection<IdentityUserRole<string>> UserRoles { get; set; } = new List<IdentityUserRole<string>>();

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

Crear `src/Domain/Entities/ApplicationRole.cs`

```cs
using Microsoft.AspNetCore.Identity;

namespace CleanArchitecture.Domain.Entities;

public class ApplicationRole : IdentityRole
{
    public ApplicationRole()
    {
    }

    public ApplicationRole(string roleName)
        : base(roleName)
    {
    }
}
```

## ApplicationUserConfiguration

Crear `src/Infrastructure/Data/Configurations/ApplicationUserConfiguration.cs`

```cs
using CleanArchitecture.Domain.Entities;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace CleanArchitecture.Infrastructure.Configurations;

public class ApplicationUserConfiguration : IEntityTypeConfiguration<ApplicationUser>
{
    public void Configure(EntityTypeBuilder<ApplicationUser> builder)
    {
        // Indexes.
        builder.HasIndex(au => au.Email)
            .IsUnique();

        builder.HasIndex(au => au.RefreshToken)
            .IsUnique();

        // Many-to-Many.
        builder.HasMany(au => au.UserRoles)
            .WithOne()
            .HasForeignKey(uc => uc.UserId)
            .IsRequired();

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

## ApplicationRoleConfiguration

Crear `src/Infrastructure/Data/Configurations/ApplicationRoleConfiguration.cs`

```cs
using CleanArchitecture.Domain.Entities;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace CleanArchitecture.Infrastructure.Configurations;

public class ApplicationRoleConfiguration : IEntityTypeConfiguration<ApplicationRole>
{
    public void Configure(EntityTypeBuilder<ApplicationRole> builder)
    {
    }
}
```

## ApplicationDbContext

Editar `src/Infrastructure/Data/ApplicationDbContext.cs` y remplazar por:

```cs
public class ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
    : IdentityDbContext<ApplicationUser, ApplicationRole, string>(options), IApplicationDbContext
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
    .AddIdentityCore<ApplicationUser>()
    .AddRoles<ApplicationRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
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

Si existe, eliminar el directorio `src/Infrastructure/Data/Migrations` que se creo al añadir [EntityFramework](./entity-framework.md)

```bash
cd src/WebApi

dotnet ef migrations add Initial -p ../Infrastructure/Infrastructure.csproj  -c ApplicationDbContext  -o ../Infrastructure/Data/Migrations

dotnet ef database update -c ApplicationDbContext
```

:::danger
**StyleCop** genera un error si se añade un `ssing` que no se utiliza, por lo que se ha de comprobar los archivos
generados en `src/Infrastructure/Data/Migrations` y eliminar los `using` que no se usen.
:::

```bash
Build started...
Build succeeded.
[23:06:17 WRN] Sensitive data logging is enabled. Log entries and exception messages may include sensitive application data; this mode should only be enabled during development.
[23:06:17 INF] Executed DbCommand (61ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE DATABASE "CleanArchitecture";
[23:06:17 INF] Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "__EFMigrationsHistory" (
    "MigrationId" character varying(150) NOT NULL,
    "ProductVersion" character varying(32) NOT NULL,
    CONSTRAINT "PK___EFMigrationsHistory" PRIMARY KEY ("MigrationId")
);
[23:06:17 INF] Executed DbCommand (19ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT EXISTS (
    SELECT 1 FROM pg_catalog.pg_class c
    JOIN pg_catalog.pg_namespace n ON n.oid=c.relnamespace
    WHERE n.nspname='public' AND
          c.relname='__EFMigrationsHistory'
)
[23:06:17 INF] Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT "MigrationId", "ProductVersion"
FROM "__EFMigrationsHistory"
ORDER BY "MigrationId";
[23:06:17 INF] Applying migration '20240516205915_Initial'.
[23:06:17 INF] Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetRoles" (
    "Id" text NOT NULL,
    "Name" character varying(256),
    "NormalizedName" character varying(256),
    "ConcurrencyStamp" text,
    CONSTRAINT "PK_AspNetRoles" PRIMARY KEY ("Id")
);
[23:06:17 INF] Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUsers" (
    "Id" text NOT NULL,
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
[23:06:17 INF] Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetRoleClaims" (
    "Id" integer GENERATED BY DEFAULT AS IDENTITY,
    "RoleId" text NOT NULL,
    "ClaimType" text,
    "ClaimValue" text,
    CONSTRAINT "PK_AspNetRoleClaims" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_AspNetRoleClaims_AspNetRoles_RoleId" FOREIGN KEY ("RoleId") REFERENCES "AspNetRoles" ("Id") ON DELETE CASCADE
);
[23:06:17 INF] Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUserClaims" (
    "Id" integer GENERATED BY DEFAULT AS IDENTITY,
    "UserId" text NOT NULL,
    "ClaimType" text,
    "ClaimValue" text,
    CONSTRAINT "PK_AspNetUserClaims" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_AspNetUserClaims_AspNetUsers_UserId" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id") ON DELETE CASCADE
);
[23:06:17 INF] Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUserLogins" (
    "LoginProvider" text NOT NULL,
    "ProviderKey" text NOT NULL,
    "ProviderDisplayName" text,
    "UserId" text NOT NULL,
    CONSTRAINT "PK_AspNetUserLogins" PRIMARY KEY ("LoginProvider", "ProviderKey"),
    CONSTRAINT "FK_AspNetUserLogins_AspNetUsers_UserId" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id") ON DELETE CASCADE
);
[23:06:17 INF] Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUserRoles" (
    "UserId" text NOT NULL,
    "RoleId" text NOT NULL,
    "ApplicationUserId" text,
    CONSTRAINT "PK_AspNetUserRoles" PRIMARY KEY ("UserId", "RoleId"),
    CONSTRAINT "FK_AspNetUserRoles_AspNetRoles_RoleId" FOREIGN KEY ("RoleId") REFERENCES "AspNetRoles" ("Id") ON DELETE CASCADE,
    CONSTRAINT "FK_AspNetUserRoles_AspNetUsers_ApplicationUserId" FOREIGN KEY ("ApplicationUserId") REFERENCES "AspNetUsers" ("Id"),
    CONSTRAINT "FK_AspNetUserRoles_AspNetUsers_UserId" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id") ON DELETE CASCADE
);
[23:06:17 INF] Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE "AspNetUserTokens" (
    "UserId" text NOT NULL,
    "LoginProvider" text NOT NULL,
    "Name" text NOT NULL,
    "Value" text,
    CONSTRAINT "PK_AspNetUserTokens" PRIMARY KEY ("UserId", "LoginProvider", "Name"),
    CONSTRAINT "FK_AspNetUserTokens_AspNetUsers_UserId" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id") ON DELETE CASCADE
);
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "IX_AspNetRoleClaims_RoleId" ON "AspNetRoleClaims" ("RoleId");
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE UNIQUE INDEX "RoleNameIndex" ON "AspNetRoles" ("NormalizedName");
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "IX_AspNetUserClaims_UserId" ON "AspNetUserClaims" ("UserId");
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "IX_AspNetUserLogins_UserId" ON "AspNetUserLogins" ("UserId");
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "IX_AspNetUserRoles_ApplicationUserId" ON "AspNetUserRoles" ("ApplicationUserId");
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "IX_AspNetUserRoles_RoleId" ON "AspNetUserRoles" ("RoleId");
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE INDEX "EmailIndex" ON "AspNetUsers" ("NormalizedEmail");
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE UNIQUE INDEX "IX_AspNetUsers_Email" ON "AspNetUsers" ("Email");
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE UNIQUE INDEX "IX_AspNetUsers_RefreshToken" ON "AspNetUsers" ("RefreshToken");
[23:06:17 INF] Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE UNIQUE INDEX "UserNameIndex" ON "AspNetUsers" ("NormalizedUserName");
[23:06:17 INF] Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
INSERT INTO "__EFMigrationsHistory" ("MigrationId", "ProductVersion")
VALUES ('20240516205915_Initial', '8.0.5');
Done.
```
