# Multi-Tenancy

## ¿Qué es?

La **multi-tenancy** (o multi-inquilino) es un modelo de arquitectura en el que una misma aplicación, desplegada una sola vez, da servicio a muchos clientes distintos —llamados *tenants* (inquilinos)— manteniendo los datos de cada uno separados e invisibles para los demás.

## ¿Por qué existe?

Imagina un edificio de apartamentos. Hay **una sola estructura** (cimientos, ascensor, tuberías, tejado) que mantiene el casero, pero cada inquilino tiene su propio piso con llave: nadie entra en el apartamento de otro. La alternativa sería construir una casa unifamiliar entera por cada inquilino: más cara de levantar y de mantener.

En software pasa lo mismo. Si tienes un producto SaaS (por ejemplo, una herramienta de facturación) y lo venden a 500 empresas, tienes dos opciones:

- **Single-tenant:** desplegar 500 copias independientes de la aplicación, una por empresa. Mantener, actualizar y monitorizar 500 instalaciones.
- **Multi-tenant:** desplegar **una** aplicación que atiende a las 500 empresas a la vez, separando sus datos por dentro.

La multi-tenancy existe para compartir la infraestructura (servidores, código, despliegues) entre muchos clientes sin que ninguno vea los datos de otro. Reduce costes y simplifica el mantenimiento: arreglas un bug una vez y se arregla para todos.

> Si vienes del mundo del *hosting*, piensa en multi-tenancy como "alojamiento compartido": muchos sitios en un mismo servidor, pero cada uno con su carpeta y sus permisos.

## ¿Cuándo y para qué se usa?

Aparece en prácticamente cualquier producto **SaaS** (Software as a Service): un CRM que usan miles de empresas, una tienda online donde cada comercio gestiona su catálogo, una app de gestión de proyectos donde cada organización ve solo sus tableros.

El caso típico: te registras en un servicio, creas una cuenta para tu empresa, invitas a tus compañeros y a partir de ahí todos veis **solo** los datos de vuestra organización. Esa organización es el *tenant*. La aplicación es la misma para ti y para cualquier otra empresa cliente, pero cada una vive en su propio compartimento.

## Lo mínimo que necesitas saber

**1. El tenant es la unidad de aislamiento**

Un tenant suele corresponder a un cliente, una empresa o una organización (no a un usuario individual). Un tenant puede tener muchos usuarios dentro.

```csharp
public class Tenant
{
    public Guid Id { get; set; }
    public string Name { get; set; }        // "Acme S.L."
    public string Subdomain { get; set; }   // "acme" → acme.miapp.com
    public bool IsActive { get; set; }
}
```

**2. Cada petición ocurre "en nombre de" un tenant**

Cuando llega una request, lo primero que hace la aplicación es averiguar *a qué tenant pertenece* (lo veremos en [Identificación del Tenant](Identificacion-del-Tenant.md)). A partir de ahí, todo lo que se lea o escriba queda acotado a ese tenant.

**3. El aislamiento puede estar en distintos niveles**

Puedes separar los datos de cada tenant en bases de datos distintas, en esquemas distintos o en las mismas tablas con una columna que marca a quién pertenece cada fila. Cada opción tiene su equilibrio entre aislamiento, coste y complejidad (ver [Estrategias de Aislamiento](Estrategias-de-Aislamiento.md)).

**4. La seguridad es lo primero**

El mayor riesgo de una app multi-tenant es la **fuga de datos entre tenants** (que la empresa A vea datos de la empresa B). Casi todo el diseño gira en torno a impedir que eso ocurra, ni siquiera por un bug.

## Lo que NO hace

- **No es lo mismo que tener varios usuarios** — muchos usuarios pueden compartir un único tenant; la multi-tenancy separa *organizaciones*, no personas.
- **No exige una base de datos por cliente** — es una de las estrategias posibles, pero a menudo se comparte la base de datos (ver estrategias).
- **No garantiza el aislamiento por sí sola** — el aislamiento lo consigues tú con el diseño; un descuido y los datos se mezclan.
- **No resuelve la personalización por cliente** — que cada tenant tenga su logo o su configuración es un problema aparte (ver [Retos Avanzados](Retos-Avanzados.md)).

---

*En resumen: multi-tenancy es servir a muchos clientes desde una sola aplicación, dándole a cada uno su propio compartimento estanco de datos.*
