# Estrategias de despliegue

**¿Qué es?** Las estrategias de despliegue son las distintas **formas de poner una nueva versión en producción** minimizando el riesgo de que los usuarios sufran cortes o errores. Definen *cómo* se sustituye la versión vieja por la nueva.

---

## ¿Por qué existe?

La forma más simple de desplegar es apagar la app, subir la versión nueva y volver a encenderla. El problema: durante ese rato la app no está disponible, y si la versión nueva falla, todos los usuarios lo notan a la vez. Estas estrategias existen para desplegar **sin cortes** y para poder **echarse atrás rápido** si algo va mal.

> Piensa en cambiar las ruedas de un coche: lo ingenuo es pararlo en mitad de la carretera; lo bueno es cambiarlas sin que los pasajeros noten nada.

---

## ¿Cuándo y para qué se usa?

En cualquier servicio que no se pueda permitir caídas: una **tienda online** en plena campaña de rebajas, una app de mensajería, un servicio de pagos. Cuanto más crítico es el servicio, más cuidado pone el equipo en cómo despliega.

---

## Lo mínimo que necesitas saber

**1. Recreate (recrear)**

La más simple: apagar la versión vieja y arrancar la nueva. Hay un breve corte de servicio. Vale para entornos internos o apps poco críticas.

**2. Rolling (despliegue progresivo)**

Si tienes varias copias de la app, las vas actualizando de una en una. Mientras una se actualiza, las demás siguen atendiendo. No hay corte, pero durante un rato conviven las dos versiones.

**3. Blue-Green (azul-verde)**

Tienes dos entornos idénticos: el "azul" (versión actual, en producción) y el "verde" (versión nueva). Despliegas y pruebas en el verde con calma; cuando está listo, rediriges todo el tráfico de golpe al verde.

```text
Usuarios ──> [AZUL v1]   (verde v2 preparándose)
Usuarios ──> [VERDE v2]  (cambio instantáneo; azul queda de reserva)
```

Si algo falla, vuelves al azul al instante.

**4. Canary (canario)**

Envías la versión nueva solo a un **pequeño porcentaje** de usuarios (p. ej. el 5%). Si todo va bien, vas subiendo el porcentaje poco a poco hasta el 100%. Si aparecen errores, solo afectan a ese grupo reducido.

> El nombre viene de los canarios que los mineros bajaban a la mina: si algo iba mal, lo detectaban antes de que afectara a todos.

**5. Rollback (vuelta atrás)**

El compañero imprescindible de todas: poder **volver a la versión anterior** rápido si la nueva da problemas. Una buena estrategia de despliegue siempre tiene un plan de rollback.

---

## Lo que NO hace

- **No arreglan el código** — reducen el *impacto* de un fallo, pero no evitan que el fallo exista.
- **No son gratis** — blue-green necesita el doble de infraestructura; canary necesita poder dividir el tráfico.
- **No sustituyen a los tests** — son la última red de seguridad, no la primera.

---

*En resumen: las estrategias de despliegue son las distintas maneras de cambiar la versión en producción sin que se note — desde el simple apagón hasta el canary que prueba con unos pocos antes de soltarlo a todos.*
