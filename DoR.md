# Definition of Ready (DoR)

_Antes de que una historia entre a desarrollo, tiene que pasar un filtro: el Definition of
Ready. Es un acuerdo del equipo sobre qué condiciones mínimas debe cumplir una historia para
considerarse "lista para trabajar". Si no las cumple, vuelve a refinamiento._

> **Nota del borrador:** esta checklist es un punto de partida propuesto para que el Grupo 1 la discuta y ajuste — es un acuerdo de equipo, así que conviene revisarla entre Yamila y Facundo antes de darla por definitiva.

---

## Checklist del equipo

| # | Ítem | Justificación (qué problema evita) |
|---|------|--------------------------------------|
| 1 | La historia tiene formato "Como [rol], quiero [acción], para [objetivo]" completo. | Evita ambigüedad sobre quién necesita qué y por qué. |
| 2 | La historia tiene al menos 2 criterios de aceptación escritos. | Sin criterios, no hay forma objetiva de saber cuándo está terminada. |
| 3 | La historia indica a qué módulo del sistema pertenece. | Facilita ubicarla dentro del alcance general y evitar solapamientos. |
| 4 | Los requisitos funcionales relacionados están identificados (ID de requisito). | Da trazabilidad entre lo que pidió el comitente y lo que se construye. |
| 5 | Se identificaron los casos límite o de error relevantes (ej. datos inválidos, token repetido). | Evita descubrir excepciones recién durante el desarrollo. |
| 6 | El equipo entiende la historia sin necesitar explicación adicional del autor. | Si hace falta explicarla oralmente, todavía no está lo bastante clara para otro integrante. |

---

## Aplicación a tres historias propias

_Aplicamos la checklist anterior a tres historias de [`docs/historias-de-usuario.md`](docs/historias-de-usuario.md)._

### Historia 1 — HU-01 Autogestión de turno

| Ítem (según checklist) | ¿Pasa? | Qué le falta (si no pasa) |
|-------------------------|--------|-----------------------------|
| 1 | Sí | — |
| 2 | Sí | — |
| 3 | Sí | — |
| 4 | Sí | — |
| 5 | Parcial | Falta definir qué pasa si el DNI tiene formato inválido o si se envía vacío. |
| 6 | Sí | — |

---

### Historia 2 — HU-02 Llamado de turnos en recepción

| Ítem (según checklist) | ¿Pasa? | Qué le falta (si no pasa) |
|-------------------------|--------|-----------------------------|
| 1 | Sí | — |
| 2 | Sí | — |
| 3 | Sí | — |
| 4 | Sí | — |
| 5 | No | Agrupa tres acciones distintas (llamar/devolver/finalizar) sin criterios de error propios para cada una; convendría separarla o sumar casos límite (ej. dos recepcionistas llamando el mismo turno a la vez). |
| 6 | Sí | — |

---

### Historia 3 — HU-04 Seguimiento remoto del turno

| Ítem (según checklist) | ¿Pasa? | Qué le falta (si no pasa) |
|-------------------------|--------|-----------------------------|
| 1 | Sí | — |
| 2 | Sí | — |
| 3 | Sí | — |
| 4 | Sí | — |
| 5 | No | Falta definir qué debe ver el paciente si el token es inválido, ya fue consumido, o si accede vía `file://` (mencionado como caso especial en la documentación funcional). |
| 6 | Sí | — |
