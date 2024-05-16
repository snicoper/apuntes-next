---
sidebar_position: 1
title: "AutoMapper"
id: auto-mapper
---

:::warning

Intentar evitar utilizar **AutoMapper** en favor de **Projections** siempre que se pueda por cuestiones de rendimiento.

:::

## Fuentes

- [AutoMapper](https://automapper.org/)

## Instalación

En el proyecto `src/Application/Application.csproj` añadir paquete `AutoMapper.Extensions.Microsoft.DependencyInjection`

Editar `src/Application/DependencyInjection.cs`.

```cs
// AutoMapper.
services.AddAutoMapper(Assembly.GetExecutingAssembly());
```

## Ejemplo de uso

Ejemplo de un `Command` que obtiene datos para editar un `User`.

```cs
public record UpdateUserCommand(string Id, string Name, string Email) : ICommand
{
    internal class Mapping : Profile
    {
        public Mapping()
        {
            CreateMap<EditUserCommand, User>();
        }
    }
}
```

En el `UpdateUserHandle.Handle`:

```cs
public async Task<Result> Handle(UpdateCategoryAbsenceCommand request, CancellationToken cancellationToken)
{
    var user = mapper.Map<User>(request)
}
```
