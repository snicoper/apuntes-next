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
