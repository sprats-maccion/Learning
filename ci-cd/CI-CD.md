# CI/CD

## ¿Qué es?

CI/CD es una forma de trabajar (y un conjunto de herramientas) que **automatiza los pasos entre escribir código y ponerlo en producción**. Las siglas significan *Continuous Integration* (Integración Continua) y *Continuous Delivery/Deployment* (Entrega/Despliegue Continuo).

## ¿Por qué existe?

Sin CI/CD, cada vez que alguien termina una funcionalidad tiene que, a mano: descargar la última versión del código, ejecutar los tests, comprobar que nada se rompe, generar la versión final (*build*) y subirla al servidor. Es lento, fácil de olvidar pasos y muy propenso a errores humanos ("en mi máquina funcionaba").

CI/CD automatiza todo eso: un robot se encarga de repetir siempre los mismos pasos, de la misma forma, cada vez que el código cambia.

> Si vienes del mundo de la cocina: CI/CD es como una cadena de montaje en un restaurante. Tú dejas los ingredientes (el código), y la cadena se encarga de cocinar, emplatar y servir siempre igual, sin que se olvide la sal.

Conviene separar las tres ideas:

- **CI (Integración Continua):** cada cambio se fusiona con el código común a menudo, y automáticamente se compila y se pasan los tests. Así los errores aparecen pronto, no semanas después.
- **CD (Entrega Continua):** además de probar, se prepara una versión lista para desplegar en cualquier momento, pero el botón final lo pulsa una persona.
- **CD (Despliegue Continuo):** el paso más allá: si todo pasa, se despliega a producción **automáticamente**, sin intervención humana.

## ¿Cuándo y para qué se usa?

Aparece en casi cualquier proyecto de software con más de una persona o que se actualice con frecuencia:

- Una **tienda online** que despliega varias veces al día sin que la web se caiga.
- Un **blog** que publica el sitio cada vez que se actualiza un artículo en el repositorio.
- Una **app de tareas** que ejecuta toda su batería de tests antes de aceptar cualquier cambio nuevo.

Su objetivo es siempre el mismo: entregar software **más rápido, más seguro y con menos errores**.

## Lo mínimo que necesitas saber

**1. Todo arranca con un cambio en el repositorio**

El flujo se dispara solo cuando ocurre algo en Git: un `push`, abrir una *pull request*, crear una etiqueta de versión... La herramienta de CI/CD está "escuchando" esos eventos.

```yaml
# Ejecutar cuando se hace push a la rama main
on:
  push:
    branches: [main]
```

**2. CI: integrar y probar**

El robot descarga el código, instala dependencias, compila y pasa los tests. Si algo falla, avisa y bloquea el cambio.

```bash
npm install      # instalar dependencias
npm run build    # compilar
npm test         # ejecutar los tests
```

**3. CD: entregar o desplegar**

Si la fase de CI pasa, se genera el artefacto final (la app empaquetada) y se publica donde toque: un servidor, un servicio en la nube, una tienda de apps...

```bash
# Ejemplo simplificado de despliegue
scp app.zip usuario@servidor:/var/www/app
ssh usuario@servidor "systemctl restart app"
```

**4. Feedback rápido**

La clave de CI/CD es enterarse pronto. Si rompes algo, te avisa en minutos (con un check rojo en la pull request), no cuando ya está en producción.

## Lo que NO hace

- **No escribe los tests por ti** — solo los ejecuta. Si no hay tests, CI no detecta nada.
- **No garantiza que el código sea bueno** — garantiza que pasa los controles que tú hayas definido.
- **No sustituye las decisiones humanas** — alguien tiene que decidir qué se prueba, cuándo se despliega y qué hacer si falla.

---

*En resumen: CI/CD es una cadena de montaje automática para tu código — integra, prueba y despliega siempre de la misma forma, para que entregar software deje de dar miedo.*
