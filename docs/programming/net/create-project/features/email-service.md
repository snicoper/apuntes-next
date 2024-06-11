---
sidebar_position: 1
title: "EmailService"
id: email-service
---

Renderiza paginas **Razor** en memoria para el envío de Email en HTML.

## Configuración

Crear `src/Application/Common/Interfaces/Emails/IEmailService.cs`

Añadir en `src/WebApi/appsettings.Development.json` la configuración para envío de Emails.

```json
"EmailSender": {
    "Host": "smtp.test.com",
    "DefaultFrom": "snicoper@test.com",
    "Username": "snicoper@test.com",
    "Password": "testPassword",
    "Port": 587,
    "UseSsl": true
}
```

Crear `src/Application/Common/Models/Settings/Settings/EmailSenderSettings.cs` para tipar las opciones de configuración.

```cs
using System.ComponentModel.DataAnnotations;

namespace CleanArchitecture.Application.Common.Models.Settings;

public class EmailSenderSettings
{
    public const string SectionName = "EmailSender";

    [Required]
    public string Host { get; set; } = default!;

    [Required]
    public string DefaultFrom { get; set; } = default!;

    [Required]
    public string Username { get; set; } = default!;

    [Required]
    public string Password { get; set; } = default!;

    [Required]
    public int Port { get; set; }

    [Required]
    public bool UseSsl { get; set; }
}
```

Editar `src/Application/DependencyInjection.cs`

Añadir parámetro `IConfiguration configuration` en `DependencyInjection.AddApplication`

```cs
public static IServiceCollection AddApplication(this IServiceCollection services, IConfiguration configuration)
{
}
```

Y añadir dentro de `AddApplication`.

```cs
services.AddOptions<EmailSenderSettings>()
    .Bind(configuration.GetSection(EmailSenderSettings.SectionName))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

Editar `src/WebApi/Program.cs` para pasar el argumento `IConfiguration configuration`:

```cs
.AddApplication(builder.Configuration)
```

## EmailViews

Constantes de las views.

Crear `src/Application/Common/Constants/EmailViews.cs`

```cs
namespace CleanArchitecture.Application.Common.Constants;

/// <summary>
/// Nombres de las vistas (cshtml) para envío de Emails.
/// </summary>
public static class EmailViews
{
    public const string TestEmail = nameof(TestEmail);
}
```

## Exceptions

Crear `src/Infrastructure/Exceptions/EmailServiceException.cs`

```cs
namespace CleanArchitecture.Infrastructure.Exceptions;

public class EmailServiceException(string message) : Exception(message);
```

## Interfaces

```cs
using System.Net.Mail;

namespace CleanArchitecture.Application.Common.Interfaces.Emails;

public interface IEmailService
{
    MailPriority MailPriority { get; set; }

    string From { get; set; }

    ICollection<string> To { get; set; }

    string? Subject { get; set; }

    string? Body { get; set; }

    bool IsBodyHtml { get; set; }

    void SendMail();

    Task SendMailWithViewAsync<TModel>(string viewName, TModel model)
        where TModel : class;
}
```

Crear `src/Application/Common/Interfaces/Views/IRazorViewToStringRendererService.cs`

```cs
namespace CleanArchitecture.Application.Common.Interfaces.Views;

public interface IRazorViewToStringRendererService
{
    Task<string> RenderViewToStringAsync(string viewName, object model, Dictionary<string, object?> viewData);
}
```

## Implementaciones

Crear `src/Infrastructure/Services/Emails/EmailService.cs`

```cs
using System.Net;
using System.Net.Mail;
using System.Text;
using CleanArchitecture.Application.Common.Interfaces.Emails;
using CleanArchitecture.Application.Common.Interfaces.Views;
using CleanArchitecture.Application.Common.Models.Settings;
using CleanArchitecture.Infrastructure.Exceptions;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;

namespace CleanArchitecture.Infrastructure.Services.Emails;

public class EmailService : IEmailService
{
    private readonly EmailSenderSettings _emailSenderSettings;
    private readonly IHostEnvironment _environment;
    private readonly ILogger<EmailService> _logger;
    private readonly IRazorViewToStringRenderer _razorViewToStringRenderer;

    public EmailService(
        IOptions<EmailSenderSettings> options,
        ILogger<EmailService> logger,
        IHostEnvironment environment,
        IRazorViewToStringRenderer razorViewToStringRenderer)
    {
        _logger = logger;
        _environment = environment;
        _razorViewToStringRenderer = razorViewToStringRenderer;
        _emailSenderSettings = options.Value;

        MailPriority = MailPriority.High;
        From = _emailSenderSettings.DefaultFrom;
        IsBodyHtml = false;
    }

    public MailPriority MailPriority { get; set; }

    public string From { get; set; }

    public ICollection<string> To { get; set; } = [];

    public string? Subject { get; set; }

    public string? Body { get; set; }

    public bool IsBodyHtml { get; set; }

    public async Task SendMailWithViewAsync<TModel>(string viewName, TModel model)
        where TModel : class
    {
        IsBodyHtml = true;
        Body = await _razorViewToStringRenderer.RenderViewToStringAsync(viewName, model, []);

        Send();
    }

    public void SendMail()
    {
        Send();
    }

    private void Send()
    {
        using var mailMessage = new MailMessage();
        mailMessage.From = new MailAddress(From);
        mailMessage.To.Add(string.Join(",", To));
        mailMessage.Subject = Subject;
        mailMessage.Body = Body;
        mailMessage.IsBodyHtml = IsBodyHtml;
        mailMessage.Priority = MailPriority;

        using var client = new SmtpClient();
        client.Host = _emailSenderSettings.Host;
        client.Port = _emailSenderSettings.Port;
        client.Credentials = new NetworkCredential(_emailSenderSettings.Username, _emailSenderSettings.Password);
        client.UseDefaultCredentials = false;
        client.EnableSsl = true;

        ValidateEmail();

        // Solo en Production se envían los mensajes por SMTP.
        if (!_environment.IsProduction())
        {
            LoggerMessage();
            return;
        }

        client.Send(mailMessage);
    }

    private void ValidateEmail()
    {
        if (To.Count == 0)
        {
            throw new EmailServiceException("The value cannot be an empty string. (Parameter 'To')");
        }

        if (string.IsNullOrEmpty(Subject))
        {
            throw new EmailServiceException("The value cannot be an empty string. (Parameter 'Subject')");
        }

        if (string.IsNullOrEmpty(Body))
        {
            throw new EmailServiceException("The value cannot be an empty string. (Parameter 'Body')");
        }
    }

    private void LoggerMessage()
    {
        var to = string.Join(", ", To);
        var body = !_environment.IsProduction() ? Body : "Body here.....";

        var stringBuilder = new StringBuilder();
        stringBuilder.Append("=========================================================\n");
        stringBuilder.Append($"From: {From}\n");
        stringBuilder.Append($"To: {to}\n");
        stringBuilder.Append($"Subject: {Subject}\n");
        stringBuilder.Append("=========================================================\n");
        stringBuilder.Append($"Body: {body}\n");
        stringBuilder.Append("=========================================================\n");

        _logger.LogDebug("{LogEmail}", stringBuilder);
    }
}
```

Crear `src/Infrastructure/Services/Views/RazorViewToStringRendererService.cs`

```cs
using CleanArchitecture.Application.Common.Interfaces.Views;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Abstractions;
using Microsoft.AspNetCore.Mvc.ModelBinding;
using Microsoft.AspNetCore.Mvc.Razor;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.AspNetCore.Mvc.ViewEngines;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Microsoft.AspNetCore.Routing;

namespace CleanArchitecture.Infrastructure.Services.Views;

public class RazorViewToStringRendererService(
        IRazorViewEngine viewEngine,
        ITempDataProvider tempDataProvider,
        IServiceProvider serviceProvider)
    : IRazorViewToStringRendererService
{
    public async Task<string> RenderViewToStringAsync(string viewName, object model, Dictionary<string, object?> viewData)
    {
        var actionContext = GetActionContext();
        var view = FindView(actionContext, viewName);

        await using var output = new StringWriter();
        var viewDataDict = new ViewDataDictionary(new EmptyModelMetadataProvider(), new ModelStateDictionary()) { Model = model };

        foreach (var data in viewData)
        {
            viewDataDict.Add(data);
        }

        TempDataDictionary tempDataDict = new(actionContext.HttpContext, tempDataProvider);

        ViewContext viewContext = new(
            actionContext,
            view,
            viewDataDict,
            tempDataDict,
            output,
            new HtmlHelperOptions());

        await view.RenderAsync(viewContext);

        return output.ToString();
    }

    private IView FindView(ActionContext actionContext, string viewName)
    {
        var getViewResult = viewEngine.GetView(null, viewName, true);
        if (getViewResult.Success)
        {
            return getViewResult.View;
        }

        var findViewResult = viewEngine.FindView(actionContext, viewName, true);
        if (findViewResult.Success)
        {
            return findViewResult.View;
        }

        var searchedLocations = getViewResult.SearchedLocations.Concat(findViewResult.SearchedLocations);
        var errorMessage = string.Join(
            Environment.NewLine,
            new[] { $"Unable to find view '{viewName}'. The following locations were searched:" }.Concat(searchedLocations));

        throw new InvalidOperationException(errorMessage);
    }

    private ActionContext GetActionContext()
    {
        var httpContext = new DefaultHttpContext { RequestServices = serviceProvider };

        return new ActionContext(httpContext, new RouteData(), new ActionDescriptor());
    }
}
```

## Views

Crear directorio en `src/Application/Common/Models/Emails`. Aquí es donde se pondran los `ViewModel`
para los correos electrónicos.

Crear `src/Application/Common/Models/Emails/TestEmailViewModel.cs` para tener el `namespace`.

```cs
namespace CleanArchitecture.Application.Common.Models.Emails;

public class TestEmailViewModel
{
}
```

Crear `src/WebApi/Views/_ViewStart.cshtml`

```html
@{
  Layout = "_Layout";
}
```

Crear `src/WebApi/Views/_ViewImports.cshtml`

```html
@using CleanArchitecture.Application.Common.Models.Emails
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

Crear `src/WebApi/Views/Shared/_Layout.cshtml`

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>@ViewData["Title"] - CleanArchitecture</title>
  <link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css"
    rel="stylesheet"
    integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN"
    crossorigin="anonymous">
</head>
<body>
<section class="container">
  <main role="main">
    @RenderBody()
  </main>
</section>
</body>
</html>
```

Crear `src/WebApi/Views/Emails/TestEmail.cshtml`

```html
@model TestEmailViewModel

@{
  ViewData["Title"] = "Aceptar invitación";
}

<h1>Contenido del email</h1>
```

Editar `src/WebApi/DependencyInjection.cs`

```cs
services.Configure<RazorViewEngineOptions>(
    options =>
    {
        options.ViewLocationFormats.Clear();
        options.ViewLocationFormats.Add("/Views/{1}/{0}" + RazorViewEngine.ViewExtension);
        options.ViewLocationFormats.Add("/Views/Emails/{0}" + RazorViewEngine.ViewExtension);
        options.ViewLocationFormats.Add("/Views/Shared/{0}" + RazorViewEngine.ViewExtension);
    });
```

Remplazar `services.AddControllers();` por `services.AddControllersWithViews()`.

```cs
services.AddControllersWithViews();
```

## Enviar un email

```cs
emailService.SendMailWithViewAsync("TestEmail", TestEmailViewModel);
```

Se le pasa el nombre de la vista `cshtml` sin la extensión y una instancia de un `ViewModel`.

Los emails en **Development** no los envía y pinta el resultado del email en consola.
