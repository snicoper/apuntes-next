---
sidebar_position: 1
title: "RequestData y ResponseData"
id: request-response-data
---

Paginación, filtros y ordenamiento de resultados de petición y respuesta.

## Instalación

Añadir `Microsoft.EntityFrameworkCore.DynamicLinq` en `src/Application/Application.csproj`

## CustomJsonSerializerOptions

Crear `src/Application/Common/Serializers/CustomJsonSerializerOptions.cs`

```cs
using System.Text.Json;

namespace CleanArchitecture.Application.Common.Serializers;

public static class CustomJsonSerializerOptions
{
    /// <summary>
    /// Opciones por defecto para <see cref="System.Text.Json" />.
    /// <para>
    /// WriteIndented = true,
    /// PropertyNameCaseInsensitive = true.
    /// </para>
    /// </summary>
    /// <returns><see cref="JsonSerializerOptions" />.</returns>
    public static JsonSerializerOptions Default()
    {
        return new JsonSerializerOptions { WriteIndented = true, PropertyNameCaseInsensitive = true };
    }
}
```

## EntityFramework Filter

### FilterOperator

Crear `src/Application/Common/EntityFramework/Filter/FilterOperator.cs`

```cs
namespace CleanArchitecture.Application.Common.EntityFramework.Filter;

public static class FilterOperator
{
    // Relational Operators.
    public const string EqualTo = "eq";
    public const string NotEqualTo = "ne";
    public const string GreaterThan = "gt";
    public const string GreaterThanOrEqual = "gte";
    public const string LessThan = "lt";
    public const string LessThanOrEqualTo = "lte";
    public const string Contains = "con";
    public const string StartsWith = "sw";
    public const string EndsWith = "ew";

    // Logical Operators.
    public const string None = " ";
    public const string And = "and";
    public const string Or = "or";

    public static string GetRelationalOperator(string op)
    {
        return op switch
        {
            EqualTo => "=",
            NotEqualTo => "!=",
            GreaterThan => ">",
            GreaterThanOrEqual => ">=",
            LessThan => " < ",
            LessThanOrEqualTo => "<=",
            Contains => ".ToLower().Contains(@{0}) ",
            StartsWith => ".ToLower().StartsWith(@{0}) ",
            EndsWith => ".ToLower().EndsWith(@{0}) ",
            _ => throw new NotImplementedException()
        };
    }

    public static string GetLogicalOperator(string op)
    {
        return op switch
        {
            None => " ",
            And => " and ",
            Or => " or ",
            _ => throw new NotImplementedException()
        };
    }
}
```

### ItemFilter

Crear `src/Application/Common/EntityFramework/Filter/ItemFilter.cs`

```cs
namespace CleanArchitecture.Application.Common.EntityFramework.Filter;

public record ItemFilter(string PropertyName, string RelationalOperator, string Value, string LogicalOperator);
```

### QueryableFilterExtensions

Crear `src/Application/Common/EntityFramework/Filter/QueryableFilterExtensions.cs`

```cs
using System.Linq.Dynamic.Core;
using System.Text;
using System.Text.Json;
using CleanArchitecture.Application.Common.Extensions;
using CleanArchitecture.Application.Common.Models;
using CleanArchitecture.Application.Common.Serializers;

namespace CleanArchitecture.Application.Common.EntityFramework.Filter;

public static class QueryableFilterExtensions
{
    public static IQueryable<TEntity> Filter<TEntity>(this IQueryable<TEntity> source, RequestData request)
    {
        if (string.IsNullOrEmpty(request.Filters))
        {
            return source;
        }

        var itemsFilter = JsonSerializer
            .Deserialize<List<ItemFilter>>(request.Filters, CustomJsonSerializerOptions.Default())?
            .ToArray() ?? [];

        if (itemsFilter.Length == 0)
        {
            return source;
        }

        var filterValues = itemsFilter
            .Select(
                filter => filter.RelationalOperator == FilterOperator.Contains
                    ? filter.Value?.ToLower()
                    : filter.Value)
            .ToDynamicArray();

        // Los FilterOperator.And se separan en un nuevo Where o dará problemas con
        // la precedencia a la hora de anidar las condiciones con los FilterOperator.Or.
        // El resto de FilterOperator, no da problemas estando en un solo Where.
        var query = new StringBuilder();

        for (var position = 0; position < itemsFilter.Length; position++)
        {
            var itemFilter = itemsFilter[position];
            var logicalOperator = !string.IsNullOrEmpty(itemFilter.LogicalOperator)
                ? FilterOperator.GetLogicalOperator(itemFilter.LogicalOperator)
                : string.Empty;

            if (logicalOperator == FilterOperator.GetLogicalOperator(FilterOperator.And))
            {
                var andQuery = new StringBuilder();
                var filter = itemsFilter[position] with { LogicalOperator = string.Empty };
                andQuery = ComposeQuery(filter, andQuery, position);

                source = source.Where(andQuery.ToString(), filterValues);
            }
            else
            {
                query = ComposeQuery(itemsFilter[position], query, position);
            }
        }

        source = source.Where(query.ToString(), filterValues);

        return source;
    }

    private static StringBuilder ComposeQuery(ItemFilter filter, StringBuilder query, int valuePosition)
    {
        var propertyName = PropertyNameToUpper(filter.PropertyName);
        var relationalOperator = FilterOperator.GetRelationalOperator(filter.RelationalOperator);

        var logicalOperator = !string.IsNullOrEmpty(filter.LogicalOperator)
            ? FilterOperator.GetLogicalOperator(filter.LogicalOperator)
            : string.Empty;

        // Comprobar si es un operador de string o lógico.
        var filterResult = filter.RelationalOperator != FilterOperator.Contains &&
                           filter.RelationalOperator != FilterOperator.StartsWith &&
                           filter.RelationalOperator != FilterOperator.EndsWith
            ? $"{logicalOperator} {propertyName} {relationalOperator} @{valuePosition}"
            : $"{logicalOperator} {string.Format(propertyName + relationalOperator, valuePosition)}";

        query.Append(filterResult);

        return query;
    }

    private static string PropertyNameToUpper(string? propertyName)
    {
        if (string.IsNullOrEmpty(propertyName))
        {
            return string.Empty;
        }

        var propertyNameParts = propertyName.Split('.');

        var propertyNameResult = propertyNameParts
            .Aggregate(string.Empty, (current, part) => current + $"{part.UpperCaseFirst()}.");

        return propertyNameResult.TrimEnd('.');
    }
}
```

## EntityFramework OrderBy

### OrderFieldEntityNotFoundException

Crear `src/Application/Common/EntityFramework/OrderBy/Exceptions/OrderFieldEntityNotFoundException.cs`

```cs
namespace CleanArchitecture.Application.Common.EntityFramework.OrderBy.Exceptions;

public class OrderFieldEntityNotFoundException(string name, object key)
    : Exception($"""Entity "{name}" ({key}) was not found for ordering.""");
```

### OrderByCommandType

Crear `src/Application/Common/EntityFramework/OrderBy/OrderByCommandType.cs`

```cs
namespace CleanArchitecture.Application.Common.EntityFramework.OrderBy;

public enum OrderByCommandType
{
    OrderBy = 1,
    ThenBy = 2
}
```

### OrderType

Crear `src/Application/Common/EntityFramework/OrderBy/OrderType.cs`

```cs
using System.ComponentModel.DataAnnotations;
using System.Text.Json.Serialization;

namespace CleanArchitecture.Application.Common.EntityFramework.OrderBy;

[JsonConverter(typeof(JsonStringEnumConverter))]
public enum OrderType
{
    [Display(Name = "None")]
    None = 0,

    [Display(Name = "ASC")]
    Asc = 1,

    [Display(Name = "DESC")]
    Desc = 2
}
```

### QueryableOrderByCommandType

Crear `src/Application/Common/EntityFramework/OrderBy/QueryableOrderByCommandType.cs`

```cs
namespace CleanArchitecture.Application.Common.EntityFramework.OrderBy;

public static class QueryableOrderByCommandType
{
    public const string OrderBy = nameof(OrderBy);
    public const string OrderByDescending = nameof(OrderByDescending);
    public const string ThenBy = nameof(ThenBy);
    public const string ThenByDescending = nameof(ThenByDescending);
}
```

### QueryableOrderByExtensions

Crear `src/Application/Common/EntityFramework/OrderBy/QueryableOrderByExtensions.cs`

```cs
using System.Linq.Dynamic.Core;
using System.Linq.Expressions;
using System.Text.Json;
using CleanArchitecture.Application.Common.EntityFramework.OrderBy.Exceptions;
using CleanArchitecture.Application.Common.Models;
using CleanArchitecture.Application.Common.Serializers;
using CleanArchitecture.Domain.Common;

namespace CleanArchitecture.Application.Common.EntityFramework.OrderBy;

public static class QueryableOrderByExtensions
{
    public static IQueryable<TEntity> Ordering<TEntity>(this IQueryable<TEntity> source, RequestData request)
    {
        if (string.IsNullOrEmpty(request.Order))
        {
            // Por defecto si existe, ordena por "Created | Id" - Descending.
            return OrderByDefault(source);
        }

        var requestItemOrderBy = JsonSerializer
            .Deserialize<RequestOrderBy>(request.Order, CustomJsonSerializerOptions.Default());

        if (requestItemOrderBy is null)
        {
            var result = OrderByDefault(source);

            return result;
        }

        source = HandleOrderByCommand(source, requestItemOrderBy, OrderByCommandType.OrderBy);

        return source;
    }

    private static IQueryable<TEntity> OrderByDefault<TEntity>(IQueryable<TEntity> source)
    {
        var propertyInfo = typeof(TEntity).GetProperty(nameof(BaseAuditableEntity.Created)) ??
                           typeof(TEntity).GetProperty(nameof(BaseEntity.Id));

        var result = propertyInfo is not null ? source.OrderBy($"{propertyInfo.Name} DESC") : source;

        return result;
    }

    private static IOrderedQueryable<TEntity> HandleOrderByCommand<TEntity>(
        IQueryable<TEntity> source,
        RequestOrderBy field,
        OrderByCommandType orderByCommandType = OrderByCommandType.ThenBy)
    {
        var fieldName = field.PropertyName.UpperCaseFirst();

        var command = orderByCommandType switch
        {
            OrderByCommandType.OrderBy => field.OrderType == OrderType.Asc
                ? QueryableOrderByCommandType.OrderByDescending
                : QueryableOrderByCommandType.OrderBy,
            OrderByCommandType.ThenBy => field.OrderType == OrderType.Desc
                ? QueryableOrderByCommandType.ThenByDescending
                : QueryableOrderByCommandType.ThenBy,
            _ => throw new NotImplementedException()
        };

        source = source.OrderByCommand(fieldName, command);
        var result = (IOrderedQueryable<TEntity>)source;

        return result;
    }

    private static IOrderedQueryable<TEntity> OrderByCommand<TEntity>(
        this IQueryable<TEntity> source,
        string orderByProperty,
        string command)
    {
        var type = typeof(TEntity);
        var orderProperty = type.GetProperty(orderByProperty);
        var properties = orderByProperty.Split('.').Select(name => typeof(TEntity).GetProperty(name)).ToArray();

        if (orderProperty is null && properties.Length > 1)
        {
            orderProperty = properties.FirstOrDefault();
        }

        if (orderProperty is null)
        {
            throw new OrderFieldEntityNotFoundException(type.Name, orderByProperty);
        }

        var parameter = Expression.Parameter(type, "p");
        var propertyAccess = Expression.MakeMemberAccess(parameter, orderProperty);
        var orderByExpression = Expression.Lambda(propertyAccess, parameter);

        var resultExpression = Expression.Call(
            typeof(Queryable),
            command,
            [type, orderProperty.PropertyType],
            source.Expression,
            Expression.Quote(orderByExpression));

        var result = (IOrderedQueryable<TEntity>)source.Provider.CreateQuery<TEntity>(resultExpression);

        return result;
    }
}
```

### RequestOrderBy

Crear `src/Application/Common/EntityFramework/OrderBy/RequestOrderBy.cs`

```cs
namespace CleanArchitecture.Application.Common.EntityFramework.OrderBy;

public record RequestOrderBy(string PropertyName, OrderType OrderType);
```

## RequestData

Crear `src/Application/Common/Models/RequestData.cs`

```cs
namespace CleanArchitecture.Application.Common.Models;

public class RequestData
{
    public int TotalItems { get; set; }

    public int PageNumber { get; set; } = 1;

    public int TotalPages { get; set; } = 1;

    public int PageSize { get; set; } = 25;

    public int Ratio { get; set; } = 2;

    public string Order { get; set; } = string.Empty;

    public string Filters { get; set; } = string.Empty;
}
```

## ResponseData

Crear `src/Application/Common/Models/ResponseData.cs`

```cs
public class ResponseData<TResponse> : RequestData
    where TResponse : class
{
    public IEnumerable<TResponse> Items { get; private init; } = new List<TResponse>();

    public bool HasPreviousPage => PageNumber > 1;

    public bool HasNextPage => PageNumber < TotalPages;

    public static async Task<ResponseData<TResponse>> CreateAsync<TEntity>(
        IQueryable<TEntity> source,
        RequestData request,
        IMapper mapper,
        CancellationToken cancellationToken)
        where TEntity : class?
    {
        var query = source
            .Filter(request)
            .Ordering(request);

        var totalItems = await query
            .CountAsync(cancellationToken);

        var items = await query
            .Skip((request.PageNumber - 1) * request.PageSize)
            .Take(request.PageSize)
            .ToListAsync(cancellationToken);

        var totalPages = (int)Math.Ceiling(items.Count / (double)request.PageSize);

        var responseData = new ResponseData<TResponse>
        {
            TotalItems = totalItems,
            PageNumber = request.PageNumber,
            TotalPages = totalPages,
            Ratio = request.Ratio,
            PageSize = request.PageSize,
            Items = mapper.Map<List<TResponse>>(items),
            Order = request.Order,
            Filters = request.Filters
        };

        return responseData;
    }
}
```

## Prueba

### Query

```cs
using CleanArchitecture.Application.Common.Interfaces.Messaging;
using CleanArchitecture.Application.Common.Security;
using CleanArchitecture.Domain.Constants;

namespace CleanArchitecture.Application.Features.Companies.Queries.GetCompanyByCurrentUser;

[Authorize(Roles = Roles.Employee)]
public record GetCompanyByCurrentUserQuery : IQuery<GetCompanyByCurrentUserResponse>;
```

### Response

```cs
namespace CleanArchitecture.Application.Features.Companies.Queries.GetCompanyByCurrentUser;

public record GetCompanyByCurrentUserResponse(string Id, string Name);
```

### Handler

```cs
using CleanArchitecture.Application.Common.Interfaces.Features.Companies;
using CleanArchitecture.Application.Common.Interfaces.Messaging;
using CleanArchitecture.Application.Common.Models;

namespace CleanArchitecture.Application.Features.Companies.Queries.GetCompanyByCurrentUser;

internal class GetCompanyByCurrentUserHandler(ICompanyService companyService)
    : IQueryHandler<GetCompanyByCurrentUserQuery, GetCompanyByCurrentUserResponse>
{
    public async Task<Result<GetCompanyByCurrentUserResponse>> Handle(
        GetCompanyByCurrentUserQuery request,
        CancellationToken cancellationToken)
    {
        var company = await companyService.GetCompanyAsync(cancellationToken);
        var result = new GetCompanyByCurrentUserResponse(company.Id, company.Name);

        return Result.Success(result);
    }
}
```
