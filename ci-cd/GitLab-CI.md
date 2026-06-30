# GitLab CI/CD

## ¿Qué es?

GitLab CI/CD es la herramienta de integración y despliegue continuo integrada en GitLab. Se configura con un único archivo, `.gitlab-ci.yml`, en la raíz del repositorio, donde defines las fases y los trabajos del pipeline.

## ¿Por qué existe?

Al igual que GitHub Actions vive dentro de GitHub, GitLab CI/CD vive dentro de GitLab. Si tu código está alojado allí, tienes CI/CD nativo sin conectar servicios externos: una sola plataforma para el repositorio, las pull requests (aquí *merge requests*) y la automatización.

> Si ya conoces GitHub Actions, piensa en GitLab CI/CD como "lo mismo, pero con un solo archivo de configuración y vocabulario propio (*stages*, *runners*, *merge requests*)".

## ¿Cuándo y para qué se usa?

En proyectos alojados en GitLab que necesiten automatizar pruebas y despliegues:

- Pasar los tests de un **blog** en cada merge request antes de aceptarlo.
- Compilar una **tienda online**, crear su imagen Docker y desplegarla en un servidor.
- Generar y publicar la documentación de una **app de tareas** automáticamente.

## Lo mínimo que necesitas saber

**1. Un solo archivo: `.gitlab-ci.yml`**

Todo el pipeline se describe en este archivo en la raíz del proyecto.

**2. Stages (fases)**

Defines las fases y su orden. Los trabajos de una fase no empiezan hasta que la anterior termina bien.

```yaml
stages:
  - build
  - test
  - deploy
```

**3. Jobs (trabajos)**

Cada job pertenece a una fase con `stage:` y ejecuta una lista de comandos en `script:`.

```yaml
ejecutar-tests:
  stage: test
  script:
    - npm install
    - npm test
```

**4. Runners**

Los *runners* son las máquinas que ejecutan los jobs. GitLab ofrece runners compartidos (en su nube) o puedes registrar los tuyos propios.

**5. Ejemplo completo mínimo**

```yaml
stages:
  - test
  - deploy

ejecutar-tests:
  stage: test
  image: node:20          # imagen Docker donde correr el job
  script:
    - npm install
    - npm test

desplegar:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main                # este job solo corre en la rama main
```

**6. Variables y secrets**

Las contraseñas se guardan en *Settings → CI/CD → Variables* y se leen como variables de entorno, sin escribirlas en el archivo.

```yaml
script:
  - echo "Desplegando con token $API_TOKEN"
```

## Lo que NO hace

- **No funciona fuera de GitLab** — está pensado para repositorios de GitLab.
- **No usa varios archivos por defecto** — toda la configuración parte de `.gitlab-ci.yml` (aunque puede incluir otros).
- **No te da runners infinitos gratis** — los runners compartidos tienen cuota de minutos según el plan.

---

*En resumen: GitLab CI/CD es el motor de pipelines nativo de GitLab — un único `.gitlab-ci.yml` describe fases y trabajos que se ejecutan solos ante cada cambio.*
