# Ejercicio: partir una épica en slices verticales

## La épica

> Como usuario de la billetera, quiero enviar dinero a otro usuario de la app para pagarle
> sin usar efectivo.

_Así como está, es una épica gorda: no se puede estimar, no se puede terminar en una
iteración, y esconde decisiones que nadie tomó todavía._

**Criterio de corte que usamos:** cada historia tiene que poder usarse sola, de punta a punta (pantalla + lógica + datos), por un usuario real. No cortamos por capas técnicas ("pantalla de envío", "API de transferencias", "tabla de movimientos"), porque ninguna de esas partes, sola, le permite a nadie pagar.

---

## Parte A — Historias verticales

### Historia 1 — Enviar dinero a un contacto con saldo suficiente

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero enviar dinero a un contacto de mi agenda que ya usa la app, teniendo saldo suficiente, para pagarle sin efectivo. |

**Criterios de aceptación**

1. Dado que tengo saldo suficiente y elijo un contacto que usa la app, cuando confirmo el envío, entonces se descuenta el monto de mi saldo y se acredita en el saldo del destinatario.
2. Dado que el envío se completó, cuando vuelvo a la pantalla principal, entonces veo el movimiento en mi historial con monto, destinatario y fecha/hora.

_Es el camino más simple de punta a punta: con esta sola historia, la app ya permite pagar._

---

### Historia 2 — Enviar dinero por alias a alguien que no está en mis contactos

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero enviar dinero a alguien que no está en mis contactos ingresando su alias, para pagarle aunque no lo tenga agendado. |

**Criterios de aceptación**

1. Dado que ingreso un alias válido, cuando lo busco, entonces el sistema me muestra el nombre del titular para confirmar antes de enviar.
2. Dado que ingreso un alias que no existe, cuando lo busco, entonces el sistema me indica "Alias no encontrado" y no me deja avanzar al monto.

---

### Historia 3 — Aviso de saldo insuficiente antes de confirmar

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero que el sistema me avise si no tengo saldo suficiente antes de confirmar el envío, para no intentar una operación que va a fallar. |

**Criterios de aceptación**

1. Dado que el monto ingresado supera mi saldo disponible, cuando intento continuar, entonces el sistema muestra "Saldo insuficiente" con el saldo disponible y no habilita el botón de confirmar.
2. Dado que corrijo el monto a un valor menor o igual a mi saldo, cuando lo modifico, entonces el aviso desaparece y se habilita el botón de confirmar.
3. Dado que el monto es exactamente igual a mi saldo, cuando intento continuar, entonces el sistema me deja confirmar y, al completarse el envío, mi saldo queda en $0.

---

### Historia 4 — Compartir el comprobante del envío

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero compartir el comprobante de un envío realizado, para demostrarle al destinatario que ya le pagué. |

**Criterios de aceptación**

1. Dado que un envío se completó, cuando presiono "Compartir comprobante", entonces se genera una imagen con monto, destinatario, fecha/hora y número de operación, y se abre el menú de compartir del celular.
2. Dado que el envío no se completó (rechazado o pendiente), cuando miro ese movimiento, entonces la opción "Compartir comprobante" no aparece.

---

### Historia 5 — Cancelar un envío antes de confirmarlo

| Campo | Detalle |
|-------|---------|
| Historia | Como usuario de la billetera, quiero poder cancelar el envío antes de confirmarlo definitivamente, para corregir un error de monto o destinatario. |

**Criterios de aceptación**

1. Dado que estoy en la pantalla de confirmación, cuando presiono "Cancelar", entonces la operación no se ejecuta, mi saldo no cambia y no queda ningún movimiento en mi historial.
2. Dado que cancelé, cuando vuelvo a la pantalla anterior, entonces el destinatario y el monto que había cargado siguen completos para poder corregirlos.

---

### Historia 6 — Recibir aviso cuando me envían dinero

| Campo | Detalle |
|-------|---------|
| Historia | Como destinatario de un envío, quiero recibir una notificación cuando alguien me transfiere dinero, para saber que ya me pagaron sin tener que revisar la app. |

**Criterios de aceptación**

1. Dado que otro usuario me envió dinero y el envío se completó, cuando se acredita en mi saldo, entonces recibo una notificación con el monto y el nombre de quien envió.
2. Dado que el envío fue rechazado o revertido, cuando termina el proceso, entonces no recibo ninguna notificación de acreditación.

---

## Parte B — Los caminos que no salen bien

**Historia elegida:** Historia 1 — Enviar dinero a un contacto con saldo suficiente

| Pregunta | Qué hace el sistema | Quién decide (analista / negocio / técnica) |
|----------|----------------------|-----------------------------------------------|
| ¿Qué pasa si el saldo es insuficiente? | No habilita el botón de confirmar y muestra "Saldo insuficiente" junto con el saldo disponible. Se resuelve antes de tocar ningún saldo (ver Historia 3). | **Analista:** es un caso que se detecta y documenta en el análisis; no requiere una decisión de negocio porque la única respuesta posible es no enviar. |
| ¿Qué pasa si el destinatario no existe o está dado de baja? | Rechaza la operación **antes de descontar el saldo** y muestra "Este destinatario no puede recibir dinero". No se permite reintentar al mismo destinatario. | **Negocio:** define qué se le informa al usuario (por privacidad no se dice si la cuenta está suspendida o cerrada) y que no hay reintento. |
| ¿Qué pasa si el sistema descuenta el saldo y falla antes de acreditarlo del otro lado? | El débito y el crédito se registran en **una única transacción**: si cualquiera de los dos falla, no se aplica ninguno. El usuario ve "El envío no se realizó. Tu saldo no cambió." No existe un estado intermedio "pendiente". | **Técnica** decide cómo se garantiza la atomicidad. **Negocio** aprobó que ante una falla se rechace el envío completo en lugar de dejarlo pendiente, porque para el usuario es peor no saber dónde está su plata que tener que reintentar. |
| ¿Qué pasa si el usuario aprieta "Enviar" dos veces? | Cada envío lleva un **número de operación único** que se genera al abrir la pantalla de confirmación. Si llega un segundo pedido con el mismo número, el servidor no lo procesa y devuelve el resultado del primero. Además, el botón se deshabilita después del primer clic. | **Técnica.** Deshabilitar el botón solo no alcanza (no cubre, por ejemplo, un reintento automático del celular por mala señal), por eso la decisión principal es el número de operación. |
| ¿Qué pasa si se cae la conexión justo después de confirmar? | La app muestra "Verificando tu envío…" y no permite volver a enviar. Al recuperar la conexión, consulta el estado por el número de operación y muestra "Envío realizado" o "El envío no se realizó. Tu saldo no cambió." Nunca reintenta sola. | **Técnica** define la consulta por número de operación; **negocio** definió el texto y que no haya reintento automático, porque un reintento sin confirmar el estado puede terminar en un pago doble. |

---

## Parte C — Defensa

_Se hace oral, en el plenario. No se documenta en este archivo._
