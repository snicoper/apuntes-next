---
sidebar_position: 1
title: "Authorize"
id: authorize
---

Autorización por **Roles** o **Policies** desde los **Commands** o **Queries** de **MediatR**

## Constants

### Policies

Crear `src/Domain/Constants/Policies.cs`

```cs
namespace CleanArchitecture.Domain.Constants;

public abstract class Policies
{
    public const string CanPurge = nameof(CanPurge);
}
```

### Roles

Crear `src/Domain/Constants/Roles.cs`

```cs
namespace CleanArchitecture.Domain.Constants;

public abstract class Roles
{
    /// <summary>
    /// Administrador del sitio.
    /// </summary>
    public const string Admin = nameof(Admin);

    /// <summary>
    /// Staff del sitio.
    /// </summary>
    public const string Staff = nameof(Staff);

    /// <summary>
    /// User del sitio.
    /// </summary>
    public const string User = nameof(User);

    /// <summary>
    /// Anónimo.
    /// </summary>
    public const string Anonymous = nameof(Anonymous);
}
```

## IUserRepository

Crear `src/Domain/Repositories/IUserRepository.cs`

```cs
using CleanArchitecture.Domain.Entities;

namespace CleanArchitecture.Domain.Repositories;

public interface IUserRepository
{
    /// <summary>
    /// Comprueba si un <see cref="ApplicationUser" /> tiene un role asignado.
    /// </summary>
    /// <param name="userId">Id de usuario.</param>
    /// <param name="role">Role a verificar.</param>
    /// <returns>True si tiene el role asignado, false en caso contrario.</returns>
    Task<bool> IsInRoleAsync(string userId, string role);

    Task<bool> AuthorizeAsync(string userId, string policyName);
}
```

Crear `src/Infrastructure/Repositories/UserRoleRepository.cs`

```cs
using CleanArchitecture.Domain.Entities;
using CleanArchitecture.Domain.Repositories;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;

namespace CleanArchitecture.Infrastructure.Repositories;

public class UserRoleRepository(
    UserManager<ApplicationUser> userManager,
    IUserClaimsPrincipalFactory<ApplicationUser> userClaimsPrincipalFactory,
    IAuthorizationService authorizationService)
    : IUserRepository
{
    public async Task<bool> IsInRoleAsync(string userId, string role)
    {
        var user = userManager
            .Users
            .SingleOrDefault(u => u.Id == userId);

        var result = user != null && await userManager.IsInRoleAsync(user, role);

        return result;
    }

    public async Task<bool> AuthorizeAsync(string userId, string policyName)
    {
        var user = userManager
            .Users
            .SingleOrDefault(u => u.Id == userId);

        if (user is null)
        {
            return false;
        }

        var principal = await userClaimsPrincipalFactory.CreateAsync(user);

        var result = await authorizationService.AuthorizeAsync(principal, policyName);

        return result.Succeeded;
    }
}
```

## AuthorizeAttribute

Crear `src/Application/Common/Security/AuthorizeAttribute.cs`

```cs
namespace CleanArchitecture.Application.Common.Security;

/// <summary>
/// Specifies the class this attribute is applied to requires authorization.
/// </summary>
[AttributeUsage(AttributeTargets.Class, AllowMultiple = true)]
public class AuthorizeAttribute : Attribute
{
    /// <summary>
    /// Initializes a new instance of the <see cref="AuthorizeAttribute" /> class.
    /// </summary>
    public AuthorizeAttribute()
    {
    }

    /// <summary>
    /// Gets or sets a comma delimited list of roles that are allowed to access the resource.
    /// </summary>
    public string Roles { get; set; } = string.Empty;

    /// <summary>
    /// Gets or sets the policy name that determines access to the resource.
    /// </summary>
    public string Policy { get; set; } = string.Empty;
}
```

## MediatR AuthorizationBehaviour

Crear `src/Application/Common/Behaviours/AuthorizationBehaviour.cs`

```cs
using System.Reflection;
using CleanArchitecture.Application.Common.Exceptions;
using CleanArchitecture.Application.Common.Interfaces.Users;
using CleanArchitecture.Application.Common.Security;
using CleanArchitecture.Domain.Repositories;
using MediatR;

namespace CleanArchitecture.Application.Common.Behaviours;

public class AuthorizationBehaviour<TRequest, TResponse>(ICurrentUserService currentUserService, IUserRepository userRepository)
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : notnull
{
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        var authorizeAttributes = request.GetType().GetCustomAttributes<AuthorizeAttribute>();
        var attributes = authorizeAttributes as AuthorizeAttribute[] ?? authorizeAttributes.ToArray();

        if (attributes.Length == 0)
        {
            return await next();
        }

        // Must be authenticated user.
        if (currentUserService.Id is null)
        {
            throw new UnauthorizedAccessException();
        }

        // Role-based authorization.
        await RoleBasedAuthorization(attributes, currentUserService.Id);

        // Policy-based authorization.
        await PolicyBasedAuthorization(attributes, currentUserService.Id);

        // User is authorized / authorization not required.
        return await next();
    }

    private async Task RoleBasedAuthorization(IEnumerable<AuthorizeAttribute> attributes, string userId)
    {
        var authorizeAttributesWithRoles = attributes.Where(a => !string.IsNullOrWhiteSpace(a.Roles));
        var attributesWithRoles = authorizeAttributesWithRoles as AuthorizeAttribute[] ?? authorizeAttributesWithRoles.ToArray();

        if (attributesWithRoles.Length != 0)
        {
            var authorized = false;

            foreach (var roles in attributesWithRoles.Select(a => a.Roles.Split(',')))
            {
                foreach (var role in roles)
                {
                    var isInRole = await userRepository.IsInRoleAsync(userId, role.Trim());

                    if (!isInRole)
                    {
                        continue;
                    }

                    authorized = true;
                    break;
                }
            }

            // Must be a member of at least one role in roles.
            if (!authorized)
            {
                throw new ForbiddenAccessException();
            }
        }
    }

    private async Task PolicyBasedAuthorization(IEnumerable<AuthorizeAttribute> attributes, string userId)
    {
        var authorizeAttributesWithPolicies = attributes.Where(a => !string.IsNullOrWhiteSpace(a.Policy));

        var attributesWithPolicies =
            authorizeAttributesWithPolicies as AuthorizeAttribute[] ?? authorizeAttributesWithPolicies.ToArray();

        if (attributesWithPolicies.Length == 0)
        {
            return;
        }

        foreach (var policy in attributesWithPolicies.Select(a => a.Policy))
        {
            var authorized = await userRepository.AuthorizeAsync(userId, policy);

            if (!authorized)
            {
                throw new ForbiddenAccessException();
            }
        }
    }
}
```

## DependencyInjection

Editar `src/Application/DependencyInjection.cs` y añadir `config.AddBehavior(typeof(IPipelineBehavior<,>), typeof(AuthorizationBehaviour<,>));`

```cs
// MediatR.
services.AddMediatR(
    config =>
    {
        config.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly());
        config.AddBehavior(typeof(IPipelineBehavior<,>), typeof(UnhandledExceptionBehaviour<,>));
        config.AddBehavior(typeof(IPipelineBehavior<,>), typeof(AuthorizationBehaviour<,>)); // <-
        config.AddBehavior(typeof(IPipelineBehavior<,>), typeof(ValidationBehaviour<,>));
    });
```

## Prueba

En los **Commands** o **Queries** añadir el `[Attribute]` con los `Roles` o `Policies` requeridos para ejecutar el `Handler`

```cs
[Authorize(Roles = Roles.Staff)]
public record CreateDepartmentCommand(string Name, string Background, string Color)
    : ICommand<string>
{
}
```
