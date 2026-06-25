# Estrategias de Aislamiento de Datos

## ¿Qué es?

Son las distintas formas de **separar físicamente los datos de cada tenant** en la capa de persistencia. Hay tres clásicas: una base de datos por tenant, un esquema por tenant dentro de una misma base, o una base de datos compartida con una columna que identifica a cada tenant.

## ¿Por qué existe?

Todo el aislamiento de una app multi-tenant se reduce a una pregunta: *¿dónde guardo los datos de cada cliente para que no se mezclen?* No hay una respuesta única. Es un equilibrio entre tres fuerzas que tiran en direcciones opuestas:

- **Aislamiento y seguridad** (¿qué pasa si hay un bug?).
- **Coste y escalabilidad** (¿cuánto cuesta tener 10.000 tenants?).
- **Complejidad operativa** (¿cuánto duele hacer una migración o una copia de seguridad?).

> Piensa en ello como vivienda: un chalet por familia (máxima privacidad, máximo coste), un piso por familia en un mismo bloque (privacidad media, coste medio) o un albergue compartido con taquillas (mínimo coste, privacidad solo "lógica").

## ¿Cuándo y para qué se usa?

Eliges la estrategia al diseñar el producto, según el tipo de cliente. Un SaaS para bancos, con requisitos legales estrictos, tenderá a una base de datos por cliente. Un producto masivo para pequeños comercios, con cientos de miles de cuentas, casi siempre comparte base de datos. Muchos productos **mezclan** estrategias: base compartida para el plan básico y base dedicada para clientes *enterprise*.

## Lo mínimo que necesitas saber

**1. Base de datos por tenant (*database-per-tenant*)**

Cada tenant tiene su propia base de datos. La aplicación elige la cadena de conexión correcta según el tenant de la petición.

```csharp
// Se resuelve la conexión a partir del tenant actual
string connectionString = tenantStore.GetConnectionString(currentTenant.Id);
using var connection = new SqlConnection(connectionString);
```

- ✅ Máximo aislamiento, copias de seguridad y restauración por cliente, fácil de borrar un tenant entero.
- ❌ Muchas bases que mantener y migrar; caro con miles de tenants.

**2. Esquema por tenant (*schema-per-tenant*)**

Una sola base de datos, pero cada tenant tiene su propio *schema* (un espacio de nombres con sus tablas: `acme.Orders`, `globex.Orders`).

```csharp
// El nombre del esquema se interpola según el tenant
string schema = currentTenant.Subdomain;          // "acme"
var sql = $"SELECT * FROM [{schema}].[Orders]";    // ojo: validar el schema antes
```

- ✅ Buen aislamiento sin pagar una base por cliente.
- ❌ Complejo de mantener y no todos los motores lo soportan con comodidad.

**3. Base de datos compartida con discriminador (*shared schema*)**

Todos los tenants comparten las mismas tablas. Cada fila lleva una columna `TenantId` que indica de quién es. Es la estrategia más común y la más barata.

```csharp
public class Order
{
    public Guid Id { get; set; }
    public Guid TenantId { get; set; }   // ← discriminador: a qué tenant pertenece
    public decimal Total { get; set; }
}
```

Aquí **toda** consulta debe filtrar por `TenantId`. Olvidarlo una sola vez = fuga de datos. Por eso se automatiza (ver [Aislamiento por Fila](Aislamiento-por-Fila.md)).

- ✅ El más barato y escalable; una sola migración para todos.
- ❌ Aislamiento solo "lógico": depende de que el código filtre bien siempre.

**4. No tiene por qué ser una elección única**

Un patrón habitual es el **modelo híbrido**: la mayoría de tenants en base compartida y los clientes grandes en base dedicada. La aplicación decide la estrategia tenant a tenant.

## Lo que NO hace

- **No existe la opción "perfecta"** — cada una sacrifica algo; eliges según tus clientes y tu presupuesto.
- **La compartida no aísla por sí sola** — sin un filtro automático por `TenantId`, los datos se mezclan a la primera.
- **Cambiar de estrategia más tarde no es trivial** — migrar de base compartida a base por tenant (o al revés) es un proyecto en sí mismo; conviene decidir pronto.

---

*En resumen: aislar tenants es elegir dónde poner la pared —entre bases de datos, entre esquemas o entre filas— sabiendo que más aislamiento cuesta más dinero y más trabajo.*
