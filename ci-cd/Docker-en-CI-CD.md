# Docker en CI/CD

**¿Qué es?** Docker es una tecnología que empaqueta una aplicación junto con todo lo que necesita para funcionar (librerías, dependencias, configuración) en una unidad portátil llamada *contenedor*. En CI/CD se usa para que el pipeline corra siempre en un entorno idéntico y predecible.

---

## ¿Por qué existe?

El clásico "en mi máquina funcionaba" ocurre porque cada ordenador tiene versiones distintas de las herramientas. Docker elimina ese problema: el contenedor lleva dentro su propia versión exacta de todo. Si funciona en el contenedor, funcionará igual en cualquier sitio donde se ejecute ese mismo contenedor.

> Si nunca lo has usado: piensa en un contenedor como una fiambrera que lleva la comida *y* los cubiertos *y* el plato. No dependes de lo que haya en la cocina de destino.

---

## ¿Cuándo y para qué se usa?

En CI/CD, Docker aparece en dos momentos clave:

- **Como entorno de ejecución del pipeline:** los jobs corren *dentro* de una imagen (p. ej. `node:20` o `python:3.12`), garantizando que todos usan las mismas versiones.
- **Como producto final:** el pipeline construye una imagen Docker de tu **tienda online** o tu **app de tareas** y la publica, lista para desplegarse en cualquier servidor que ejecute contenedores.

---

## Lo mínimo que necesitas saber

**1. La imagen base define el entorno**

Casi todas las herramientas de CI/CD dejan elegir en qué imagen corre el job.

```yaml
test:
  image: node:20        # el job corre dentro de un contenedor con Node 20
  script:
    - npm test
```

**2. El Dockerfile describe tu imagen**

Para empaquetar tu propia app defines un `Dockerfile`: una receta de cómo construir su contenedor.

```dockerfile
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

**3. Construir y publicar la imagen en el pipeline**

Un paso típico de CD: construir la imagen y subirla a un registro (un almacén de imágenes).

```bash
docker build -t miapp:1.0 .
docker push registro.ejemplo.com/miapp:1.0
```

**4. Desplegar = arrancar el contenedor**

En el servidor de destino, desplegar es simplemente descargar la imagen y arrancarla. No hay que instalar dependencias a mano.

---

## Lo que NO hace

- **No es obligatorio en CI/CD** — puedes tener pipelines sin Docker, pero te pierdes la reproducibilidad del entorno.
- **No es una máquina virtual completa** — comparte el núcleo del sistema, por eso es mucho más ligero.
- **No despliega solo** — Docker empaqueta y ejecuta; orquestar muchos contenedores en producción es trabajo de otras herramientas (como Kubernetes).

---

*En resumen: Docker da a tu pipeline un entorno idéntico de principio a fin — empaqueta la app con todas sus dependencias para que "en mi máquina funcionaba" deje de ser un problema.*
