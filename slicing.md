# Ejercicio: partir una épica en slices verticales

## La épica

> Como usuario de la billetera, quiero enviar dinero a otro usuario de la app para pagarle
> sin usar efectivo.

_Así como está, es una épica gorda: no se puede estimar, no se puede terminar en una
iteración, y esconde decisiones que nadie tomó todavía._

> **Nota del borrador:** este ejercicio es genérico (billetera), no del proyecto Fertya — es un ejercicio de práctica de slicing. Este es un intento de respuesta para que el Grupo 1 lo revise y ajuste antes de la defensa oral.

---

## Parte A — Historias verticales

### Historia 1 — Enviar dinero a un contacto con saldo suficiente

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero enviar dinero a un contacto de mi agenda que ya usa la app, teniendo saldo suficiente, para pagarle sin efectivo. |

**Criterios de aceptación**

1. Dado que tengo saldo suficiente y elijo un contacto válido, cuando confirmo el envío, entonces se descuenta el monto de mi saldo y se acredita en el saldo del destinatario.
2. Dado que el envío se completó, cuando vuelvo a la pantalla principal, entonces veo el movimiento reflejado en mi historial.

---

### Historia 2 — Enviar dinero por alias o CVU/CBU a un contacto nuevo

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero enviar dinero a alguien que no está en mis contactos ingresando su alias, para pagarle aunque no lo tenga agendado. |

**Criterios de aceptación**

1. Dado que ingreso un alias válido, cuando lo busco, entonces el sistema me muestra el nombre del destinatario para confirmar antes de enviar.
2. Dado que confirmo el envío, cuando se procesa, entonces el destinatario queda disponible para un envío más rápido la próxima vez.

---

### Historia 3 — Ver el estado de saldo insuficiente antes de intentar enviar

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero que el sistema me avise si no tengo saldo suficiente antes de confirmar el envío, para no intentar una operación que va a fallar. |

**Criterios de aceptación**

1. Dado que el monto ingresado supera mi saldo disponible, cuando intento continuar, entonces el sistema me lo indica y no me deja avanzar a confirmar.

---

### Historia 4 — Recibir confirmación y comprobante del envío

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero recibir una confirmación clara (en pantalla) de que el envío se realizó, para tener la certeza de que la operación se completó. |

**Criterios de aceptación**

1. Dado que el envío se procesó con éxito, cuando termina la operación, entonces veo una pantalla de confirmación con monto, destinatario y fecha/hora.

---

### Historia 5 — Cancelar un envío en curso antes de confirmar

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero poder cancelar el envío antes de confirmarlo definitivamente, para corregir un error de monto o destinatario. |

**Criterios de aceptación**

1. Dado que estoy en la pantalla de confirmación, cuando presiono "Cancelar", entonces la operación no se ejecuta y vuelvo a la pantalla anterior sin cambios en mi saldo.

---

## Parte B — Los caminos que no salen bien

**Historia elegida:** Historia 1 — Enviar dinero a un contacto con saldo suficiente

| Pregunta | Qué hace el sistema | Quién decide (analista / negocio / técnica) |
|----------|----------------------|-----------------------------------------------|
| ¿Qué pasa si el saldo es insuficiente? | No permite confirmar el envío y muestra un aviso claro del faltante. | Analista (ya cubierto en Historia 3). |
| ¿Qué pasa si el destinatario no existe o está dado de baja? | Rechaza la operación antes de descontar saldo y muestra un mensaje indicando que el destinatario no es válido. | Negocio (define si se permite reintentar o contactar soporte). |
| ¿Qué pasa si el sistema descuenta el saldo y falla antes de acreditarlo del otro lado? | La operación debe tratarse como transacción atómica: si falla la acreditación, se revierte el débito automáticamente (o queda en un estado "pendiente" a reconciliar). | Técnica (decisión de arquitectura/transaccionalidad). |
| ¿Qué pasa si el usuario aprieta "Enviar" dos veces? | El sistema debe identificar la segunda pulsación como duplicado (ej. bloqueando el botón tras el primer click, o con un identificador de operación) y no procesar el envío dos veces. | Técnica. |
| ¿Qué pasa si se cae la conexión justo después de confirmar? | Al recuperar conexión, el usuario debe poder ver el estado real de la operación (si se concretó o no) en vez de quedar en duda; no se debe reintentar automáticamente sin confirmar el estado previo. | Técnica, con impacto en negocio (afecta la confianza del usuario). |

---

## Parte C — Defensa

_Se hace oral, en el plenario. No se documenta en este archivo._
