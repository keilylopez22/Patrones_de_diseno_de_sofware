---

### 1. Factory Method
**Estado: Aplicable — No implementado**

**¿Qué es?**  
Define una interfaz para crear objetos, pero deja que las subclases decidan qué clase instanciar.

**¿Dónde aplicaría en el proyecto?**  
En la creación de respuestas de reservaciones según su estado (`Recibida`, `Confirmada`, `Cancelada`, `Pagada`). Actualmente en `ReservacionesController` y el servicio correspondiente, la lógica de construcción de respuesta está mezclada con la lógica de negocio.

**Ejemplo**  
Se podría crear un `ReservacionResponseFactory` que, dado el estado de la reservación, retorne un objeto de respuesta con formato y campos distintos.

**¿Por qué no está?**  
El proyecto usa funciones anónimas directas con `.Select(r => new { ... })` en los servicios.

---

### 2. Abstract Factory
**Estado: Aplicable — No implementado**

**¿Qué es?**  
Sirve para crear familias de objetos relacionados sin especificar sus clases concretas.

**¿Dónde aplicaría en el proyecto?**  
En la capa de servicios. El proyecto ya tiene interfaces (`ICabanaService`, `IClienteService`, `IEmpleadoService`, etc.) registradas en `Program.cs`, pero no existe una fábrica abstracta que agrupe la creación de estas familias de servicios.

**Ejemplo**  
Una `IServiceFactory` que agrupe la creación de servicios relacionados con reservaciones (`IReservacionService`, `ICabanaService`, `IClienteService`) para facilitar pruebas y sustitución de implementaciones.

**¿Por qué no está?**  
El proyecto delega la creación al contenedor de inyección de dependencias de ASP.NET (`AddScoped`), el ASP lo hace de manera implicita

---

### 3. Builder
**Estado: Aplicable — No implementado**

**¿Qué es?**  
Crea una clase es un constructor especialista en objetos complejos

**¿Dónde aplicaría en el proyecto?**  
En la construcción de `RegistroReservacion`. Este objeto requiere validar cliente, cabaña, empleado, fechas y calcular `TotalPagar`. Actualmente toda esa lógica está dispersa en el servicio de reservaciones.

**Ejemplo**  
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
Se crea una nueva instancia copiando otra.

**¿Dónde aplicaría en el proyecto?**  
Tendría uso marginal. El caso más cercano sería clonar una `RegistroReservacion` se puede crear paquetes como un paqueta para una luna de miel.

**Limitación:**  
valle escondido no ofrece paquetes.

**Conclusión:** Aplicable solo si se implementa la funcionalidad de reservaciones de modo paquete como el de la luna de miel.

---

### 5. Singleton
**Estado: Ya aplicado (implícitamente por el framework)**

**¿Qué es?**  
Solo puede existir un objeto de una sola clase .

**¿Dónde está aplicado?**  
El framework ASP.NET Core aplica este patrón de forma implícita en las conexiones a las bases de datos

**¿Está implementado de forma explícita en código propio?**  
No, porque puede volver mas lento el sistema y dar problemas de concurrencia.

---

### 6. Adapter
**Estado: Aplicable — No **

**¿Qué es?**  
Convierte la interfaz de una clase en otra que el cliente espera, permitiendo que clases incompatibles trabajen juntas.

**¿Dónde aplicaría en el proyecto?**  
En la conversión entre entidades de base de datos y DTOs. Actualmente la transformación se hace manualmente dentro de cada servicio con código repetitivo:

```csharp
// En PersonaService.cs — conversión manual repetida
.Select(p => new PersonaResponseDto {
    IdPersona = p.IdPersona,
    Nombres = p.Nombres,
    ...
})
```

**Ejemplo**  
Un `PersonaAdapter` o `ReservacionAdapter` que encapsule la conversión entre `Persona` ↔ `PersonaResponseDto` ↔ `PersonaCreateDto`, evitando duplicar la lógica de mapeo en cada servicio.


---

### 7. Bridge
**Estado: Aplicable**

**¿Qué es?**  
hace que un objeto pueda evolucionar por partes.





---

### 8. Composite
**Estado:No implementado**

**¿Qué es?**  
Compone objetos en estructuras de árbol para representar jerarquías parte-todo. Permite tratar objetos individuales y composiciones de forma uniforme.

**¿Dónde aplicaría en el proyecto?**  
El caso más cercano sería en `ReservacionServicio` (tabla que relaciona reservaciones con múltiples servicios adicionales). Una reservación podría verse como un componente compuesto de: cabaña + servicios adicionales + cliente.



---


