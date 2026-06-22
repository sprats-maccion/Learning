# Capa de Infraestructura

## ¿Qué es?

La capa de infraestructura es el **borde externo** de Clean Architecture: contiene los detalles técnicos concretos de tu aplicación. Aquí viven las implementaciones reales que hablan con la base de datos, envían emails, llaman a APIs externas o leen ficheros. Es donde el negocio se conecta con el mundo real.

## ¿Por qué existe?

El dominio y los casos de uso definen *qué necesitan* mediante interfaces ("necesito guardar un pedido", "necesito cobrar un importe"), pero alguien tiene que cumplir esas promesas con tecnología de verdad. Ese es el papel de la infraestructura.

Existe para concentrar en un solo lugar todo lo que es "detalle reemplazable": el motor de base de datos, la librería de envío de correo, la pasarela de pago. Al estar aislada en el borde, puedes cambiar cualquiera de esas piezas sin tocar el centro de la aplicación.

> Si piensas en tu aplicación como una cocina, el dominio es la receta y la infraestructura son los electrodomésticos: puedes cambiar el horno por otro modelo sin reescribir la receta.

## ¿Cuándo y para qué se usa?

Se usa para implementar todo lo que cruza la frontera de la aplicación hacia fuera. En una tienda online, aquí estaría el repositorio que guarda pedidos en SQL Server, el cliente que llama a la API de la pasarela de pago y el servicio que manda el email de confirmación.

Es la capa que más se toca cuando cambia la tecnología y la que **menos** debería contener lógica de negocio: idealmente, solo traduce entre el contrato del centro y la herramienta concreta.

## Lo mínimo que necesitas saber

**1. Implementa las interfaces que define el centro**

```csharp
// La interfaz vive en el centro; la implementación, aquí.
public class RepositorioPedidosSql(string cadenaConexion) : IRepositorioPedidos
{
    public async Task Guardar(Pedido pedido)
    {
        using var conexion = new SqlConnection(cadenaConexion);
        // ... INSERT / UPDATE real en la base de datos
    }
}
```

**2. Aquí —y solo aquí— viven los detalles técnicos**

Cadenas de conexión, SQL, llamadas HTTP, rutas de fichero, claves de API. Nada de esto debe filtrarse hacia el dominio o los casos de uso.

**3. Traduce entre el mundo exterior y el modelo de negocio**

La base de datos devuelve filas y columnas; la infraestructura las convierte en entidades de dominio que el resto de la aplicación entiende.

```csharp
var fila = await LeerDeBaseDeDatos(id);
var pedido = new Pedido(fila.Id, fila.ClienteId); // de "datos crudos" a entidad
```

**4. Se conecta al resto al arrancar la aplicación**

En la configuración (la capa más externa) se indica qué implementación usar para cada interfaz. Cambiar de tecnología es cambiar esa línea.

```csharp
services.AddScoped<IRepositorioPedidos, RepositorioPedidosSql>();
```

## Lo que NO hace

- **No contiene reglas de negocio** — esas viven en el dominio; aquí solo hay traducción y detalles técnicos.
- **No es conocida por el centro** — el dominio y los casos de uso ignoran que esta capa existe; solo conocen las interfaces.
- **No debería tomar decisiones de negocio** — si te ves escribiendo un `if` con una regla importante dentro de un repositorio, esa lógica está en el sitio equivocado.

---

*En resumen: la capa de infraestructura es el borde técnico que cumple, con tecnología real, las promesas que el centro define mediante interfaces — todo lo reemplazable vive aquí, y solo aquí.*
