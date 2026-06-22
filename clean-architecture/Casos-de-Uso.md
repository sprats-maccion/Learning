# Casos de Uso

## ¿Qué es?

Un caso de uso es una clase que representa **una acción concreta que tu aplicación sabe hacer**: "registrar un cliente", "confirmar un pedido", "cancelar una reserva". Vive en la capa de aplicación, justo alrededor del dominio, y se encarga de **orquestar** los pasos de esa acción.

## ¿Por qué existe?

Las entidades del dominio conocen sus propias reglas, pero una acción real suele necesitar **coordinar varias cosas**: leer datos, llamar a una entidad, guardar el resultado, enviar un aviso. Si ese baile se escribe dentro de un controlador web, la lógica de la aplicación queda atrapada en el framework y es imposible de reutilizar o probar.

Los casos de uso existen para tener esa coordinación en un sitio independiente del exterior. Cada caso de uso responde a la pregunta "¿qué pasos hay que dar para realizar esta acción?", apoyándose en el dominio para las reglas y en interfaces para todo lo demás.

> Si ya conoces el patrón "servicio de aplicación", un caso de uso es justo eso, pero acotado a **una sola acción** en lugar de agrupar muchas en un único servicio gigante.

## ¿Cuándo y para qué se usa?

Hay un caso de uso por cada cosa que un usuario (o el sistema) puede pedir a la aplicación. En una tienda online: `AñadirProductoAlCarrito`, `ConfirmarPedido`, `AplicarDescuento`. En un blog: `PublicarArtículo`, `ModerarComentario`.

Son el punto de entrada a la lógica: la capa web recibe la petición HTTP, la traduce y llama al caso de uso. El caso de uso hace el trabajo y devuelve un resultado simple, sin saber que vino de la web.

## Lo mínimo que necesitas saber

**1. Un caso de uso = una acción**

```csharp
public class ConfirmarPedido(IRepositorioPedidos repositorio)
{
    public async Task Ejecutar(int pedidoId)
    {
        var pedido = await repositorio.ObtenerPorId(pedidoId);
        pedido.Confirmar();                  // la regla vive en la entidad
        await repositorio.Guardar(pedido);
    }
}
```

**2. Orquesta, pero no contiene las reglas**

El caso de uso decide *el orden* de los pasos (leer → confirmar → guardar), pero la regla de "no confirmar un pedido vacío" sigue dentro de la entidad `Pedido`. El caso de uso coordina; el dominio decide.

**3. Depende de interfaces, no de tecnología**

Para guardar, cobrar o avisar, el caso de uso usa interfaces definidas en el centro. No sabe si por detrás hay SQL, un email real o un objeto falso de test.

```csharp
public class ProcesarCompra(IPasarelaPago pasarela, IRepositorioPedidos repositorio)
{
    public async Task Ejecutar(Pedido pedido, string tarjeta)
    {
        if (await pasarela.Cobrar(pedido.Total, tarjeta))
            await repositorio.Guardar(pedido);
    }
}
```

**4. Recibe y devuelve datos simples (DTOs)**

Para no acoplarse a la web, el caso de uso trabaja con objetos sencillos de entrada y salida, no con peticiones HTTP ni modelos de pantalla.

```csharp
public record DatosRegistro(string Nombre, string Email);
```

## Lo que NO hace

- **No contiene reglas de negocio profundas** — esas viven en el dominio; el caso de uso solo las invoca en el orden correcto.
- **No conoce la web ni la base de datos** — habla con el exterior solo a través de interfaces.
- **No gestiona detalles de transporte** — códigos de estado HTTP, rutas o serialización JSON son trabajo de la capa de adaptadores, no suyo.

---

*En resumen: un caso de uso orquesta los pasos de una acción concreta de la aplicación, apoyándose en el dominio para las reglas y en interfaces para el mundo exterior — una acción, una clase.*
