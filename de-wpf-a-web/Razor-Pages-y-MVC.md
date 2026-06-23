# Razor Pages y MVC

## ¿Qué es?

Razor Pages y MVC son dos formas de hacer aplicaciones web con ASP.NET Core en las que
**el servidor genera el HTML** y se lo manda ya terminado al navegador. **Razor** es el
lenguaje de plantillas que mezcla HTML con C# para construir esas páginas.

## ¿Por qué existe?

Es la forma más directa de pasar de WPF a la web sin aprender JavaScript ni montar un
frontend aparte. Tú sigues escribiendo C# y una plantilla parecida a XAML (Razor), y el
servidor te devuelve la página lista. Encaja muy bien con aplicaciones donde cada acción
implica navegar a otra página: paneles de gestión, formularios, catálogos, informes.

> Si vienes de WPF: una página Razor se parece a un `.xaml` con su code-behind. El
> archivo `.cshtml` es la "vista" (como tu XAML) y el archivo `.cshtml.cs` es el
> code-behind, donde manejas lo que pasa al cargar la página o al enviar un formulario.

## ¿Cuándo y para qué se usa?

Para aplicaciones web "clásicas" donde el servidor manda la página completa: el alta de
un cliente en un sistema de gestión, un catálogo de productos con su buscador, el panel
interno de una empresa. Si la app no necesita mucha interactividad instantánea ni una
API para móviles, Razor Pages suele ser la opción más sencilla y rápida.

**Razor Pages vs. MVC** (la diferencia, en una frase): Razor Pages organiza el código
**por página** (cada página con su lógica al lado, más simple para empezar); MVC lo
organiza **por responsabilidades** separando *Controllers*, *Models* y *Views* (más
estructura, mejor para aplicaciones grandes). Ambos usan Razor y conviven en el mismo
proyecto.

## Lo mínimo que necesitas saber

**1. Una vista Razor es HTML con C# incrustado**

El `@` cambia de HTML a C#. Puedes meter bucles y condiciones para generar HTML dinámico:

```cshtml
<h1>Productos</h1>
<ul>
@foreach (var producto in Model.Productos)
{
    <li>@producto.Nombre — @producto.Precio €</li>
}
</ul>
```

**2. Una página tiene su modelo (su "code-behind")**

En Razor Pages, el archivo `.cshtml.cs` prepara los datos que la vista mostrará. El
método `OnGet` se ejecuta al abrir la página (como un `Loaded` de WPF):

```csharp
public class ProductosModel : PageModel
{
    private readonly IProductoRepository _repo;
    public List<Producto> Productos { get; set; } = new();

    public ProductosModel(IProductoRepository repo) => _repo = repo;

    public async Task OnGetAsync()
    {
        Productos = await _repo.ObtenerTodosAsync(); // se inyecta el repo (ver Inyección de dependencias)
    }
}
```

**3. Los formularios se envían al servidor y se procesan en `OnPost`**

En lugar de un evento `Click`, un formulario HTML hace un `POST` a tu página, y tú lo
recoges en `OnPostAsync`:

```cshtml
<form method="post">
    <input asp-for="NuevoProducto.Nombre" />
    <button type="submit">Guardar</button>
</form>
```

```csharp
public async Task<IActionResult> OnPostAsync()
{
    await _repo.CrearAsync(NuevoProducto);
    return RedirectToPage("Productos"); // recarga la lista
}
```

**4. En MVC, un controlador devuelve una vista**

El equivalente con MVC reparte el trabajo: el *Controller* recibe la petición, prepara
el *Model* y elige la *View* a renderizar:

```csharp
public class ProductosController : Controller
{
    public async Task<IActionResult> Index()
    {
        var productos = await _repo.ObtenerTodosAsync();
        return View(productos); // renderiza Views/Productos/Index.cshtml
    }
}
```

**5. La página viaja ya renderizada como HTML**

A diferencia de una API, aquí el cliente recibe HTML listo para mostrar, no datos en
bruto. El navegador solo tiene que dibujarlo.

## Lo que NO hace

- **No da interactividad instantánea por sí solo** — para actualizar la pantalla sin recargar necesitas [JavaScript](JavaScript-y-el-Navegador.md) o [Blazor](Blazor.md).
- **No sirve como backend para una app móvil** — para eso necesitas una [Web API](Web-API-y-REST.md) que devuelva datos, no HTML.
- **No ejecuta C# en el navegador** — todo el C# de Razor corre en el servidor; al navegador solo llega HTML.

---

*En resumen: Razor Pages y MVC generan el HTML en el servidor mezclando C# y plantillas Razor; es el camino más corto desde WPF a la web cuando cada acción implica navegar a otra página.*
