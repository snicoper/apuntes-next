---
sidebar_position: 1
title: "Localización"
id: localization
---

## Fuentes

- [Globalization and localization in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/localization)

## Configuración

Crear `src/Application/Common/Constants/AppCultures.cs`

```cs
using System.Globalization;

namespace CleanArchitecture.Application.Common.Constants;

public static class AppCultures
{
    /// <summary>
    /// Cultura por defecto.
    /// </summary>
    public static readonly CultureInfo DefaultCulture = GetDefaultCulture();

    /// <summary>
    /// Culturas soportadas.
    /// </summary>
    private static readonly CultureInfo Es = new("es");

    private static readonly CultureInfo EsEs = new("es-ES");

    private static readonly CultureInfo En = new("en");

    private static readonly CultureInfo EnUs = new("en-US");

    /// <summary>
    /// Obtener todas las culturas disponibles.
    /// </summary>
    /// <returns>Lista CultureInfo disponibles.</returns>
    public static List<CultureInfo> GetAll()
    {
        return [Es, EsEs, En, EnUs];
    }

    /// <summary>
    /// Obtener cultura por defecto.
    /// </summary>
    private static CultureInfo GetDefaultCulture()
    {
        return EsEs;
    }
}
```

## Localization y Resource

Crear `src/Application/Common/Localization/SharedResource.cs`

```cs
namespace CleanArchitecture.Application.Common.Localization;

public class SharedResource
{
}
```

Crear `src/Application/Common/Resources/Common/Localization/SharedResource.resx`

```xml
<?xml version="1.0" encoding="utf-8"?>

<root>
  <xsd:schema id="root" xmlns="" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata">
    <xsd:element name="root" msdata:IsDataSet="true">

    </xsd:element>
  </xsd:schema>
  <resheader name="resmimetype">
    <value>text/microsoft-resx</value>
  </resheader>
  <resheader name="version">
    <value>1.3</value>
  </resheader>
  <resheader name="reader">
    <value>System.Resources.ResXResourceReader, System.Windows.Forms, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089</value>
  </resheader>
  <resheader name="writer">
    <value>System.Resources.ResXResourceWriter, System.Windows.Forms, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089</value>
  </resheader>
</root>
```

Crear `src/Application/Common/Resources/Common/Localization/SharedResource.es-ES.resx`

```xml
<?xml version="1.0" encoding="utf-8"?>

<root>
  <xsd:schema id="root" xmlns="" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata">
    <xsd:element name="root" msdata:IsDataSet="true">

    </xsd:element>
  </xsd:schema>
  <resheader name="resmimetype">
    <value>text/microsoft-resx</value>
  </resheader>
  <resheader name="version">
    <value>1.3</value>
  </resheader>
  <resheader name="reader">
    <value>System.Resources.ResXResourceReader, System.Windows.Forms, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089</value>
  </resheader>
  <resheader name="writer">
    <value>System.Resources.ResXResourceWriter, System.Windows.Forms, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089</value>
  </resheader>
</root>
```

Crear `src/Application/Common/Resources/Common/Localization/SharedResource.en-US.resx`

```xml
<?xml version="1.0" encoding="utf-8"?>

<root>
  <xsd:schema id="root" xmlns="" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata">
    <xsd:element name="root" msdata:IsDataSet="true">

    </xsd:element>
  </xsd:schema>
  <resheader name="resmimetype">
    <value>text/microsoft-resx</value>
  </resheader>
  <resheader name="version">
    <value>1.3</value>
  </resheader>
  <resheader name="reader">
    <value>System.Resources.ResXResourceReader, System.Windows.Forms, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089</value>
  </resheader>
  <resheader name="writer">
    <value>System.Resources.ResXResourceWriter, System.Windows.Forms, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089</value>
  </resheader>
</root>
```

## DependencyInjection

Editar `src/WebApi/DependencyInjection.cs`

En la parte de `services.AddControllersWithViews()` añadir:

```cs
services.AddControllersWithViews()
    .AddViewLocalization(LanguageViewLocationExpanderFormat.Suffix)
    .AddDataAnnotationsLocalization(
        options =>
        {
            options.DataAnnotationLocalizerProvider = (_, factory) => factory.Create(typeof(SharedResource));
        });
```

Y añadir:

```cs
// Culture.
services.Configure<RequestLocalizationOptions>(
    options =>
    {
        options.DefaultRequestCulture = new RequestCulture(AppCultures.DefaultCulture, AppCultures.DefaultCulture);
        options.SupportedCultures = AppCultures.GetAll();
        options.SupportedUICultures = AppCultures.GetAll();
    });

// Localization.
services.AddLocalization(options => { options.ResourcesPath = "Common/Resources"; });
```

Editar `src/WebApi/Program.cs` añadir:

```cs
app.UseRequestLocalization();
```

## Controller

Crear `src/WebApi/Controllers/LocalizationsController.cs`

```cs
using CleanArchitecture.Application.Common.Models;
using CleanArchitecture.WebApi.Infrastructure;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace CleanArchitecture.WebApi.Controllers;

[Route("api/v{version:apiVersion}/localizations")]
public class LocalizationsController : ApiControllerBase
{
    /// <summary>
    /// Obtener el locale actual.
    /// </summary>
    /// <returns>Locale actual.</returns>
    [AllowAnonymous]
    [HttpGet("current-locale")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<ActionResult<Result<CurrentLocaleResponse>>> CurrentLocale()
    {
        var result = await Sender.Send(new CurrentLocaleQuery());

        return result;
    }

    /// <summary>
    /// Obtener locales soportados.
    /// </summary>
    /// <returns>Lista de locales soportadas.</returns>
    [AllowAnonymous]
    [HttpGet("supported-locales")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<ActionResult<Result<SupportedLocalesResponse>>> SupportedLocales()
    {
        var result = await Sender.Send(new SupportedLocalesQuery());

        return result;
    }
}
```

## CurrentLocaleQuery

Crear `src/Application/Features/Localizations/Queries/CurrentLocale/CurrentLocaleHandler.cs`

```cs
using CleanArchitecture.Application.Common.Interfaces.Messaging;
using CleanArchitecture.Application.Common.Models;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Localization;

namespace CleanArchitecture.Application.Localizations.Queries.CurrentLocale;

internal class CurrentLocaleHandler(IHttpContextAccessor httpContextAccessor)
    : IQueryHandler<CurrentLocaleQuery, CurrentLocaleResponse>
{
    public Task<Result<CurrentLocaleResponse>> Handle(CurrentLocaleQuery request, CancellationToken cancellationToken)
    {
        var culture = httpContextAccessor.HttpContext?.Features.Get<IRequestCultureFeature>();
        var locale = culture?.RequestCulture.Culture.ToString();
        var currentLocaleResponse = new CurrentLocaleResponse(locale);

        return Task.FromResult(Result.Success(currentLocaleResponse));
    }
}
```

Crear `src/Application/Localizations/Queries/CurrentLocale/CurrentLocaleQuery.cs`

```cs
using CleanArchitecture.Application.Common.Interfaces.Messaging;

namespace CleanArchitecture.Application.Localizations.Queries.CurrentLocale;

public record CurrentLocaleQuery : IQuery<CurrentLocaleResponse>;
```

Crear `src/Application/Localizations/Queries/CurrentLocale/CurrentLocaleResponse.cs`

```cs
namespace CleanArchitecture.Application.Localizations.Queries.CurrentLocale;

public record CurrentLocaleResponse(string? Locale);
```

## SupportedLocalesQuery

Crear `src/Application/Localizations/Queries/SupportedLocales/SupportedLocalesHandler.cs`

```cs
using CleanArchitecture.Application.Common.Constants;
using CleanArchitecture.Application.Common.Interfaces.Messaging;
using CleanArchitecture.Application.Common.Models;

namespace CleanArchitecture.Application.Localizations.Queries.SupportedLocales;

internal class SupportedLocalesHandler : IQueryHandler<SupportedLocalesQuery, SupportedLocalesResponse>
{
    public Task<Result<SupportedLocalesResponse>> Handle(SupportedLocalesQuery request, CancellationToken cancellationToken)
    {
        var supportedCultures = AppCultures
            .GetAll()
            .Select(cultureInfo => cultureInfo.Name)
            .ToList();

        var supportedLocalesResponse = new SupportedLocalesResponse(supportedCultures);

        return Task.FromResult(Result.Success(supportedLocalesResponse));
    }
}
```

Crear `src/Application/Localizations/Queries/SupportedLocales/SupportedLocalesQuery.cs`

```cs
using CleanArchitecture.Application.Common.Interfaces.Messaging;

namespace CleanArchitecture.Application.Localizations.Queries.SupportedLocales;

public record SupportedLocalesQuery : IQuery<SupportedLocalesResponse>;
```

Crear `src/Application/Localizations/Queries/SupportedLocales/SupportedLocalesResponse.cs`

```cs
namespace CleanArchitecture.Application.Localizations.Queries.SupportedLocales;

public record SupportedLocalesResponse(ICollection<string> Locales);
```
