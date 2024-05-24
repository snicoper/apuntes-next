---
sidebar_position: 1
title: "SOLID"
id: solid
---

## Single Responsibility Principle

El principio de responsabilidad única (**SRP**) es una regla de diseño de software que establece que cada clase o módulo debe tener solo una razón para cambiar, es decir, solo debe tener una única responsabilidad. Esto significa que una clase o módulo solo debe tener una única función o propósito dentro del sistema.

Al seguir el principio de responsabilidad única, se pueden lograr varias ventajas, como una mejor mantenibilidad del código, una mayor reutilización de código, una mejor prueba y una mejora en la legibilidad y comprensión del código.

En resumen, el principio de responsabilidad única es una regla de diseño que ayuda a crear código más modular, mantenible y fácil de entender.

## Open/Closed Principle

El principio abierto/cerrado (**OCP**) es una regla de diseño de software que establece que las entidades de software (clases, módulos, etc.) deben estar abiertas para su extensión, pero cerradas para su modificación. En otras palabras, se debe poder agregar nueva funcionalidad sin modificar el código existente.

Esto se logra a través de la utilización de herencia y polimorfismo. Al extender una clase existente en lugar de modificarla directamente, se puede agregar nueva funcionalidad sin afectar el código existente que ya está en funcionamiento.

El principio abierto/cerrado es importante porque permite el desarrollo de software más flexible y modular. Al seguir este principio, se pueden agregar nuevas funcionalidades sin tener que modificar el código existente, lo que reduce el riesgo de errores y facilita las actualizaciones y mejoras en el sistema.

En resumen, el principio abierto/cerrado es una regla de diseño que promueve la extensibilidad y la modularidad en el código, permitiendo agregar nuevas funcionalidades sin modificar el código existente.

## Liskov Substitution Principle

El principio de sustitución de Liskov (**LSP**) es una regla de diseño de software que establece que los objetos de una clase derivada deben poder ser sustituidos por objetos de su clase base sin alterar el comportamiento esperado del programa. En otras palabras, los objetos de una clase derivada deben ser capaces de reemplazar a los objetos de su clase base sin causar errores o comportamientos inesperados.

El **LSP** se basa en el principio de herencia y se aplica a las relaciones de herencia entre clases. Es importante seguir este principio porque permite la reutilización de código y la creación de estructuras modulares y escalables.

Al aplicar el **LSP**, se deben cumplir las siguientes condiciones:

1. **Precedencia**: Los objetos de la clase derivada deben ser capaces de ser sustituidos por los objetos de la clase base sin perder ninguna funcionalidad.
2. **Respeto de los contratos**: Los objetos de la clase derivada deben cumplir con los mismos contratos o comportamientos que los objetos de su clase base.
3. **Subtipo invariante**: Si una clase derivada tiene un invariante específico, los objetos de esa clase derivada también deben cumplir con ese invariante.

## Interface Segregation Principle

El principio de segregación de la interfaz (**ISP**) es una regla de diseño de software que establece que las interfaces deberían ser específicas y no genéricas para cada cliente. En otras palabras, se deben crear interfaces más pequeñas y cohesivas que se centren en las necesidades específicas de cada cliente, en lugar de tener una única interfaz amplia que cumpla con las necesidades de todos los clientes.

El **ISP** se basa en el principio de responsabilidad única y promueve la creación de interfaces más cohesivas y específicas, lo que facilita la comprensión y el mantenimiento del código. Al seguir este principio, se pueden evitar interfaces grandes y genéricas que contienen muchas funcionalidades no utilizadas por algunos clientes, lo que reduce la complejidad y el acoplamiento en el sistema.

Al aplicar el **ISP**, se deben cumplir las siguientes condiciones:

1. **Identificación de interfaces**: Se deben identificar las interfaces específicas necesarias para cada cliente.
2. **Separación de interfaces:** Se deben separar las interfaces en pequeñas partes que se centren en una funcionalidad específica.

## Dependency Inversion Principle

El principio de inversión de dependencia (**DIP**) es una regla de diseño de software que establece que las entidades de alto nivel no deben depender de las entidades de bajo nivel. En otras palabras, se debe invertir la dependencia entre las clases o componentes de un sistema para que en lugar de depender directamente de otras clases, dependen de abstracciones o interfaces.

El **DIP** se basa en el principio de separación de preocupaciones y promueve la creación de sistemas más modulares y flexibles. Al invertir la dependencia, se logra una mayor independencia y reutilización de código, ya que las clases de alto nivel solo dependen de abstracciones y no de implementaciones concretas.

Al aplicar el **DIP**, se deben cumplir las siguientes condiciones:

1. **Dependencia en abstracciones**: Las clases de alto nivel deben depender de abstracciones o interfaces en lugar de depender directamente de clases concretas.
2. **Inversión de control**: El control de la creación y configuración de objetos debe ser invertido, lo que significa que las clases de alto nivel no deben crear objetos de bajo nivel directamente, sino que deben recibirlos como dependencias a través de su constructor o métodos.
