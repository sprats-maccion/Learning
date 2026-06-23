# Routing y Middleware

## ¿Qué es?

El **routing** (enrutado) es el mecanismo que decide qué trozo de tu código atiende cada
petición, según su URL y su verbo. El **middleware** es la cadena de pasos por la que
pasa toda petición antes y después de llegar a tu código (autenticación, logging,
manejo de errores...).

## ¿Por qué existe?

En WPF, "qué código se ejecuta" lo decidía el evento: la usuaria pulsaba un botón y se
disparaba su `Click`. En la web no hay botones físicos en el servidor: lo que llega es
una petición HTTP con una URL como `/productos/42`. El routing es lo que traduce esa URL
en "ejecuta este método". Y como toda petición necesita pasos comunes (¿está autenticada?
¿lo registramos? ¿hubo un error?), el middleware permite encadenar esos pasos una sola
vez para todas las peticiones, en vez de repetirlos en cada sitio.

> Si vienes de WPF: el routing es como el sistema que mapea cada evento a su manejador,
> pero usando URLs en lugar de controles. El middleware se parece a una tubería de
> filtros por la que pasa todo antes de llegar a tu lógica.

## ¿Cuándo y para qué se usa?

El routing aparece en cuanto defines más de una página o endpoint: `/`, `/productos`,
`/productos/42`, `/carrito`. El middleware se usa para todo lo transversal: comprobar que
quien pide algo ha iniciado sesión, registrar cada petición en el log, capturar
excepciones para devolver una página de error decente, o servir archivos estáticos como
imágenes y hojas de estilo.

## Lo mínimo que necesitas saber

**1. El routing empareja URL + verbo con tu código**

```csharp
// Una petición GET a /productos/42 acaba aquí, con id = 42
app.MapGet("/productos/{id}", (int id) => $"Producto {id}");
```

En estilos con controladores, lo mismo se expresa con atributos sobre los métodos:

```csharp
[HttpGet("productos/{id}")]
public IActionResult Detalle(int id) { /* ... */ }
```

**2. Las partes variables de la URL se llaman parámetros de ruta**

En `/productos/{id}`, `{id}` es un hueco que se rellena con lo que venga en la URL y se
pasa a tu método. Es como recibir un argumento, pero sacado de la dirección.

**3. El middleware es una tubería: el orden importa**

Cada petición atraviesa los middlewares en el orden en que los declaras. El típico
arranque define esa tubería:

```csharp
app.UseExceptionHandler("/error"); // 1. captura errores de todo lo que viene después
app.UseStaticFiles();              // 2. sirve imágenes, CSS, JS
app.UseRouting();                  // 3. decide qué endpoint corresponde
app.UseAuthentication();           // 4. ¿quién eres?
app.UseAuthorization();            // 5. ¿puedes hacer esto?
app.MapControllers();              // 6. ejecuta tu código
```

> Regla práctica: la autenticación va **antes** que la autorización (primero sé quién
> eres, luego compruebo si tienes permiso), y el manejo de errores va al principio para
> que envuelva todo lo demás.

**4. Cada middleware puede dejar pasar, modificar o cortar la petición**

Por ejemplo, el middleware de autorización puede cortar la cadena y devolver un `401`
sin que tu código llegue a ejecutarse. Así centralizas reglas comunes en un solo sitio.

## Lo que NO hace

- **El routing no valida los datos** — solo decide qué método se ejecuta; validar el contenido es trabajo tuyo o de otra capa.
- **El middleware no es lógica de negocio** — es para temas transversales (seguridad, logs, errores), no para las reglas de tu dominio.
- **El orden no es opcional** — colocar un middleware en el sitio equivocado puede romper la seguridad o el manejo de errores.

---

*En resumen: el routing traduce cada URL en una llamada a tu código, y el middleware es la tubería ordenada de pasos comunes por la que pasa toda petición —seguridad, logs y errores— antes y después de tu lógica.*
