# Ejercicio: partir una épica en slices verticales

## La épica

> Como usuario de la billetera, quiero enviar dinero a otro usuario de la app para pagarle
> sin usar efectivo.

_Así como está, es una épica gorda: no se puede estimar, no se puede terminar en una
iteración, y esconde decisiones que nadie tomó todavía._

---

## Parte A — Historias verticales

_Entre 5 y 8 historias VERTICALES. Vertical significa que cada historia, sola, entrega algo
usable de punta a punta ("diseñar la pantalla de envío" no es vertical; "enviar dinero a un
contacto de la agenda con saldo suficiente" sí lo es)._

### Historia 1 — [Nombre]

| Campo | Detalle |
|-------|---------|
| Historia | Como [rol], quiero [acción], para [objetivo]. |

**Criterios de aceptación**

1. 
2. 

---

### Historia 2 — [Nombre]

| Campo | Detalle |
|-------|---------|
| Historia | Como [rol], quiero [acción], para [objetivo]. |

**Criterios de aceptación**

1. 
2. 

---

## Parte B — Los caminos que no salen bien

_Elijan UNA de las historias de la Parte A. Las últimas tres preguntas son las importantes:
para cada una, indiquen qué debería hacer el sistema y quién tendría que decidirlo._

**Historia elegida:** [Nombre / ID]

| Pregunta | Qué hace el sistema | Quién decide (analista / negocio / técnica) |
|----------|----------------------|-----------------------------------------------|
| ¿Qué pasa si el saldo es insuficiente? | | |
| ¿Qué pasa si el destinatario no existe o está dado de baja? | | |
| ¿Qué pasa si el sistema descuenta el saldo y falla antes de acreditarlo del otro lado? | | |
| ¿Qué pasa si el usuario aprieta "Enviar" dos veces? | | |
| ¿Qué pasa si se cae la conexión justo después de confirmar? | | |

---

## Parte C — Defensa

_Se hace oral, en el plenario. No se documenta en este archivo._
