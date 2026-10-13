# Definition of Ready (DoR)

_Antes de que una historia entre a desarrollo, tiene que pasar un filtro: el Definition of
Ready. Es un acuerdo del equipo sobre qué condiciones mínimas debe cumplir una historia para
considerarse "lista para trabajar". Si no las cumple, vuelve a refinamiento._

---

## Checklist del equipo

| # | Ítem | Justificación (qué problema evita) |
|---|------|--------------------------------------|
| 1 | La historia tiene formato "Como [rol], quiero [acción], para [objetivo]" con los tres componentes completos. | Evita historias sin destinatario o sin propósito, donde no se sabe a quién se le entrega valor. |
| 2 | La historia tiene al menos 2 criterios de aceptación escritos en formato "Dado / cuando / entonces". | Sin criterios no hay forma objetiva de saber cuándo está terminada; el formato obliga a explicitar la situación, la acción y el resultado. |
| 3 | Cada requisito relacionado está citado por su ID (RF-XX / RNF-XX) y ese ID existe en `requisitos.md`. | Da trazabilidad entre lo que pidió Fertya y lo que se construye; detecta historias que no se apoyan en ningún requisito. |
| 4 | Al menos un criterio de aceptación describe un caso de error o límite (dato inválido, acción repetida, falta de conexión, acción simultánea). | Evita modelar solo el camino feliz y descubrir las excepciones recién al programar o, peor, en la recepción de Fertya. |
| 5 | Las dependencias con otras historias están escritas (qué historia tiene que existir antes), o se indica explícitamente "sin dependencias". | Evita planificar una historia que no se puede probar porque falta otra; hace visible el orden de trabajo. |
| 6 | Si la historia está alcanzada por un RNF (tiempo, seguridad, usabilidad), al menos un criterio de aceptación incluye su valor medible (ej. "en menos de 2 segundos"). | Evita que los RNF queden como enunciados sueltos que nadie verifica al dar la historia por terminada. |
| 7 | La historia describe una sola acción del usuario; si tiene más de una (ej. llamar, devolver y finalizar), se divide antes de entrar. | Historias con varias acciones no se pueden estimar ni cerrar por partes, y esconden criterios de error de cada acción. |
| 8 | Cada fila de la validación INVEST tiene una observación escrita que no se limita a repetir "sí" o "cumple". | Una casilla vacía o un "sí" automático indica que el criterio no se pensó; es justo lo que la validación intenta evitar. |

---

## Aplicación a tres historias propias

_Aplicamos la checklist a tres historias tal como estaban en la **primera versión del 27/08/2026** (commit "Completo README, integrantes, requisitos, HU..."), antes de cualquier corrección. Al final se indica qué cambiamos a partir de este análisis._

### Historia 1 — HU-01 Autogestión de turno (versión 27/08)

| Ítem (según checklist) | ¿Pasa? | Qué le falta (si no pasa) |
|-------------------------|--------|-----------------------------|
| 1 | Sí | — |
| 2 | Sí | — |
| 3 | Sí | — |
| 4 | No | El único caso "de error" era un motivo desconocido, que en la app no puede ocurrir porque el motivo se elige de una lista. Faltaba qué pasa con un DNI inválido o vacío. |
| 5 | Sí | Es la primera historia del circuito; no depende de otra. |
| 6 | No | Estaba alcanzada por RNF-04 (uso sin ayuda) pero ningún criterio tenía un valor medible de tiempo. |
| 7 | Sí | — |
| 8 | Parcial | Las observaciones de "Estimable" y "Pequeña" eran genéricas ("cabe en una iteración corta"), no decían por qué. |

---

### Historia 2 — HU-02 Llamado de turnos en recepción (versión 27/08)

| Ítem (según checklist) | ¿Pasa? | Qué le falta (si no pasa) |
|-------------------------|--------|-----------------------------|
| 1 | Sí | — |
| 2 | Sí | — |
| 3 | Sí | — |
| 4 | No | Ningún criterio cubría errores: faltaba el caso de dos recepcionistas llamando el mismo turno a la vez (PB y 4º piso). |
| 5 | Parcial | La dependencia con HU-01 estaba escrita en INVEST, pero marcada como "Independiente: Sí", lo que se contradice. |
| 6 | No | El llamado tiene que verse en la pantalla de sala en tiempo real (RNF-01), pero no había un criterio con tiempo medible. |
| 7 | No | Agrupaba tres acciones (llamar, devolver y finalizar). El propio INVEST lo marcaba como "Pequeña: Parcial". |
| 8 | No | La fila "Estimable" estaba vacía. |

---

### Historia 3 — HU-04 Seguimiento remoto del turno (versión 27/08)

| Ítem (según checklist) | ¿Pasa? | Qué le falta (si no pasa) |
|-------------------------|--------|-----------------------------|
| 1 | Sí | — |
| 2 | Sí | — |
| 3 | Sí | — |
| 4 | No | Faltaba qué ve el paciente si el token es inválido, falta en la URL o ya fue consumido. |
| 5 | No | Depende del token que genera HU-01, pero no estaba escrito (fila "Independiente" sin observación). |
| 6 | No | Estaba alcanzada por RNF-01 (tiempo real) sin valor medible en ningún criterio. |
| 7 | Sí | — |
| 8 | No | Cuatro de las seis filas de INVEST estaban vacías o solo decían "Sí". |

---

## Qué cambiamos a partir de la DoR

Ninguna de las tres historias pasaba la checklist completa. A partir de este análisis:

- **08/09/2026 — commit "Cierro Requisitos y HU…":** se dividió HU-02 en HU-02a (llamar), HU-02b (devolver) y HU-02c (finalizar) (ítem 7); se agregó a HU-01 el criterio de DNI inválido (ítem 4); se agregó a HU-02a el caso de llamado simultáneo (ítem 4) y a HU-04 los casos de token inválido y ya consumido (ítem 4).
- **Revisión final:** se agregaron valores medibles de los RNF en los criterios de aceptación (ítem 6), se escribieron las dependencias en la fila "Independiente" de cada historia (ítem 5) y se completaron todas las observaciones INVEST (ítem 8). Además se corrigió el criterio de HU-01 sobre "motivo desconocido", que no reflejaba el comportamiento real de la app, y el de HU-04 sobre el token consumido, que no se bloquea.
