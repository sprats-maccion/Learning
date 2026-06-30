# Pipeline

## ¿Qué es?

Un *pipeline* (tubería) es la **secuencia de pasos automáticos** que tu código recorre desde que cambia hasta que llega a producción. Es la "receta" concreta que ejecuta una herramienta de CI/CD.

## ¿Por qué existe?

Necesitas una forma de **describir** qué pasos hay que ejecutar, en qué orden y bajo qué condiciones. El pipeline es esa descripción: normalmente un archivo de texto que vive junto a tu código y dice "primero compila, luego prueba, luego despliega".

> Si ya conoces las recetas de cocina: el pipeline es la receta escrita paso a paso. La herramienta de CI/CD es el cocinero que la sigue al pie de la letra cada vez.

Tenerlo como archivo de texto (lo que se llama *pipeline as code*) tiene una ventaja enorme: el proceso de despliegue queda versionado en Git, igual que el código. Puedes ver quién lo cambió, revertirlo si se rompe y revisarlo en una pull request.

## ¿Cuándo y para qué se usa?

Cada vez que configuras CI/CD, defines un pipeline. Los pasos cambian según el proyecto:

- En una **app de tareas** en JavaScript: instalar dependencias → linter → tests → build → desplegar.
- En una **tienda online** en .NET: restaurar paquetes → compilar → tests → crear imagen Docker → publicar.
- En un **blog** estático: generar el sitio → subirlo a un hosting.

El pipeline da estructura y nombre a cada una de esas fases.

## Lo mínimo que necesitas saber

**1. Jobs y steps (trabajos y pasos)**

Un pipeline se organiza en *jobs* (trabajos), y cada job tiene varios *steps* (pasos). Un step es una orden concreta; un job agrupa pasos relacionados.

```yaml
jobs:
  test:                          # un job llamado "test"
    steps:
      - run: npm install         # paso 1
      - run: npm test            # paso 2
```

**2. Stages o fases (orden y dependencias)**

Los jobs se agrupan en fases que se ejecutan en orden. La fase de despliegue no empieza hasta que la de tests termina bien.

```yaml
stages:
  - build
  - test
  - deploy   # solo si build y test han ido bien
```

**3. Ejecución en paralelo y en serie**

Los pasos que dependen unos de otros van en serie (uno tras otro). Los independientes pueden ir en paralelo para ahorrar tiempo (p. ej. probar en Windows y Linux a la vez).

**4. Si un paso falla, el pipeline se detiene**

Por defecto, en cuanto un paso devuelve error, el pipeline se para y marca la ejecución como fallida. Así no se despliega código roto.

**5. Artefactos**

El resultado de un paso (la app compilada, un informe de tests) se puede guardar como *artefacto* y pasárselo al siguiente paso o descargarlo después.

```yaml
artifacts:
  paths:
    - dist/        # guarda la carpeta compilada
```

## Lo que NO hace

- **No es una herramienta** — es la *definición* de los pasos; quien la ejecuta es la herramienta de CI/CD (GitHub Actions, GitLab CI, Jenkins...).
- **No decide la lógica de negocio** — solo ejecuta órdenes; lo que hace cada orden lo defines tú.
- **No se ejecuta solo** — necesita un disparador (un `push`, una pull request, una hora programada...).

---

*En resumen: el pipeline es la receta paso a paso de tu CI/CD — un archivo versionado que describe cómo tu código pasa de un commit a estar en producción.*
