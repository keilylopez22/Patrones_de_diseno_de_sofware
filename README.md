# Patrones_de_diseno_de_sofware
Analis del proyecto de hidden valley como aplicar patrones 

# Análisis de Patrones de Diseño — Hidden Valley API

> Proyecto: Sistema de reservaciones Hidden Valley  
> Tecnología: .NET 9 Web API + Blazor + PostgreSQL  
> Fecha de análisis: 2025

---

## Resumen ejecutivo

| # | Patrón | Estado |
|---|--------|--------|
| 1 | Factory Method | Aplicable — No implementado |
| 2 | Abstract Factory | Aplicable — No implementado |
| 3 | Builder | Aplicable — No implementado |
| 4 | Prototype | Aplicabilidad limitada — No implementado |
| 5 | Singleton | Ya aplicado (implícitamente por el framework) |
| 6 | Adapter | Aplicable — No implementado explícitamente |
| 7 | Bridge | Aplicable — Parcialmente presente |
| 8 | Composite | Aplicabilidad limitada — No implementado |

---

## Análisis detallado

---

### 1. Factory Method
**Estado: Aplicable — No implementado**

**¿Qué es?**  
Define una interfaz para crear objetos, pero deja que las subclases decidan qué clase instanciar.

**¿Dónde aplicaría en el proyecto?**  
En la creación de respuestas de reservaciones según su estado (`Recibida`, `Confirmada`, `Cancelada`, `Pagada`). Actualmente en `ReservacionesController` y el servicio correspondiente, la lógica de construcción de respuesta está mezclada con la lógica de negocio.

**Ejemplo concreto:**  
Se podría crear un `ReservacionResponseFactory` que, dado el estado de la reservación, retorne un objeto de respuesta con formato y campos distintos.

**¿Por qué no está?**  
El proyecto usa proyecciones anónimas directas con `.Select(r => new { ... })` en los servicios, sin abstraer la creación de objetos.

---

### 2. Abstract Factory
**Estado: Aplicable — No implementado**

**¿Qué es?**  
Provee una interfaz para crear familias de objetos relacionados sin especificar sus clases concretas.

**¿Dónde aplicaría en el proyecto?**  
En la capa de servicios. El proyecto ya tiene interfaces (`ICabanaService`, `IClienteService`, `IEmpleadoService`, etc.) registradas en `Program.cs`, pero no existe una fábrica abstracta que agrupe la creación de estas familias de servicios.

**Ejemplo concreto:**  
Una `IServiceFactory` que agrupe la creación de servicios relacionados con reservaciones (`IReservacionService`, `ICabanaService`, `IClienteService`) para facilitar pruebas y sustitución de implementaciones.

**¿Por qué no está?**  
El proyecto delega la creación al contenedor de inyección de dependencias de ASP.NET (`AddScoped`), lo cual es funcional pero no implementa el patrón de forma explícita.

---

### 3. Builder
**Estado: Aplicable — No implementado**

**¿Qué es?**  
Separa la construcción de un objeto complejo de su representación, permitiendo crear distintas representaciones con el mismo proceso.

**¿Dónde aplicaría en el proyecto?**  
En la construcción de `RegistroReservacion`. Este objeto requiere validar cliente, cabaña, empleado, fechas y calcular `TotalPagar`. Actualmente toda esa lógica está dispersa en el servicio de reservaciones.

**Ejemplo concreto:**  
Un `ReservacionBuilder` con métodos encadenados:
```
new ReservacionBuilder()
    .ConCliente(idCliente)
    .ConCabana(idCabana)
    .ConFechas(entrada, salida)
    .ConEmpleado(idEmpleado)
    .Build();
```

**¿Por qué no está?**  
La construcción del objeto se hace directamente con inicializadores de objeto (`new RegistroReservacion { ... }`) dentro del servicio.

---

### 4. Prototype
**Estado: Aplicabilidad limitada — No implementado**

**¿Qué es?**  
Permite crear nuevos objetos copiando (clonando) una instancia existente.

**¿Dónde aplicaría en el proyecto?**  
Tendría uso marginal. El caso más cercano sería clonar una `RegistroReservacion` existente para crear una reservación recurrente o una "reservación plantilla" para grupos que repiten visitas.

**Limitación:**  
El dominio del proyecto (turicentro con reservaciones únicas por fechas) no tiene un caso de uso fuerte para clonar entidades. Las entidades están ligadas a IDs de BD y fechas específicas, lo que hace que un clon siempre requiera modificaciones significativas.

**Conclusión:** Aplicable solo si se implementa la funcionalidad de reservaciones recurrentes o plantillas.

---

### 5. Singleton
**Estado: Ya aplicado (implícitamente por el framework)**

**¿Qué es?**  
Garantiza que una clase tenga una única instancia y provee un punto de acceso global a ella.

**¿Dónde está aplicado?**  
El framework ASP.NET Core aplica este patrón de forma implícita en varios niveles:

- **`ApplicationDbContext`** — registrado como `AddDbContext` (scoped, una instancia por request HTTP, equivalente funcional al Singleton dentro del scope).
- **`WebApplication` / `IConfiguration`** — instancia única durante toda la vida de la aplicación.
- **Middleware pipeline** — construido una sola vez en `Program.cs` con `app.UseSwagger()`, `app.UseCors()`, etc.

**¿Está implementado de forma explícita en código propio?**  
No. No existe ninguna clase propia con el patrón Singleton implementado manualmente (constructor privado + instancia estática). El proyecto confía completamente en el DI container del framework.

---

### 6. Adapter
**Estado: Aplicable — No implementado explícitamente**

**¿Qué es?**  
Convierte la interfaz de una clase en otra que el cliente espera, permitiendo que clases incompatibles trabajen juntas.

**¿Dónde aplicaría en el proyecto?**  
En la conversión entre entidades de dominio y DTOs. Actualmente la transformación se hace manualmente dentro de cada servicio con código repetitivo:

```csharp
// En PersonaService.cs — conversión manual repetida
.Select(p => new PersonaResponseDto {
    IdPersona = p.IdPersona,
    Nombres = p.Nombres,
    ...
})
```

**Ejemplo concreto:**  
Un `PersonaAdapter` o `ReservacionAdapter` que encapsule la conversión entre `Persona` ↔ `PersonaResponseDto` ↔ `PersonaCreateDto`, evitando duplicar la lógica de mapeo en cada servicio.

**Nota:** Librerías como AutoMapper implementan este patrón. El proyecto no usa ninguna.

---

### 7. Bridge
**Estado: Aplicable — Parcialmente presente**

**¿Qué es?**  
Desacopla una abstracción de su implementación para que ambas puedan variar independientemente.

**¿Dónde está parcialmente presente?**  
La separación entre **Interfaces** (`ICabanaService`, `IPersonaService`, etc.) y **Servicios** (`CabanaService`, `PersonaService`, etc.) es la base del patrón Bridge. La abstracción (interfaz) está desacoplada de la implementación concreta (servicio).

```
IPersonaService  ←→  PersonaService
ICabanaService   ←→  CabanaService
IEmpleadoService ←→  EmpleadoService
```

**¿Por qué solo parcialmente?**  
El patrón Bridge completo implicaría que la abstracción también pueda variar (por ejemplo, tener `IReservacionManager` que use `ICabanaService` internamente y pueda intercambiar implementaciones en tiempo de ejecución). Actualmente los controladores dependen directamente de las interfaces de servicio sin una capa de abstracción adicional.

**¿Qué faltaría para completarlo?**  
Una capa de abstracción de alto nivel (por ejemplo, `IReservacionManager`) que use las interfaces de servicio como "implementación", permitiendo intercambiar comportamientos sin modificar los controladores.

---

### 8. Composite
**Estado: Aplicabilidad limitada — No implementado**

**¿Qué es?**  
Compone objetos en estructuras de árbol para representar jerarquías parte-todo. Permite tratar objetos individuales y composiciones de forma uniforme.

**¿Dónde aplicaría en el proyecto?**  
El caso más cercano sería en `ReservacionServicio` (tabla que relaciona reservaciones con múltiples servicios adicionales). Una reservación podría verse como un componente compuesto de: cabaña + servicios adicionales + cliente.

**Limitación:**  
El dominio actual no tiene jerarquías parte-todo profundas. La relación `Reservacion → Servicios` es una simple colección, no una estructura de árbol recursiva que justifique el patrón Composite.

**Conclusión:** Aplicable solo si el negocio crece hacia paquetes de servicios anidados (ej: "Paquete Familiar" que contiene "Paquete Básico" + servicios adicionales).

---

## Conclusiones

### Patrones que aportarían mayor valor inmediato

1. **Builder** — La construcción de `RegistroReservacion` es el caso más complejo del proyecto y se beneficiaría directamente de este patrón.
2. **Adapter** — Eliminaría el código de mapeo repetitivo entre entidades y DTOs presente en todos los servicios.
3. **Bridge** — Completar la separación ya iniciada con las interfaces de servicio mejoraría la testabilidad del sistema.

### Patrones ya presentes (aunque implícitos)

- **Singleton** — Gestionado por el DI container de ASP.NET Core.
- **Bridge** — Parcialmente implementado con la separación `Interfaces/` + `Services/`.

### Patrones con baja prioridad para el dominio actual

- **Prototype** — Solo útil si se implementan reservaciones recurrentes.
- **Composite** — Solo útil si se implementan paquetes de servicios anidados.
