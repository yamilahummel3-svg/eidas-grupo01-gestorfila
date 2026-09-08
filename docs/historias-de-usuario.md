# Historias de usuario

_Presentar al menos una historia de usuario representativa por módulo._
_Cada historia debe incluir formato clásico, criterios de aceptación y validación INVEST._

---

## HU-01 — Autogestión de turno

| Campo | Detalle |
|-------|---------|
| Historia | Como paciente, quiero registrarme en la fila ingresando mi DNI y motivo de visita, para obtener un turno sin necesidad de hacer fila física en recepción. |
| Módulo | Autorecepción |
| Requisitos relacionados | RF-01, RF-01b, RF-02, RF-03, RF-04 |

### Criterios de aceptación

1. Dado que el paciente completa DNI y motivo de visita, cuando envía el formulario, entonces el sistema genera un código visible (ej. `C-001`) y lo muestra en pantalla junto a un código QR.
2. Dado que el motivo ingresado no coincide con ninguna categoría conocida, cuando se genera el turno, entonces se le asigna la categoría por defecto (`C`).
3. Dado que se genera el turno, cuando se crea en Firestore, entonces se le asocia un token único para el seguimiento posterior.
4. Dado que el DNI ingresado no es numérico o no tiene entre 7 y 8 dígitos, cuando el paciente intenta enviar el formulario, entonces el sistema rechaza el envío y muestra un aviso de formato inválido, sin crear el turno.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Sí | No depende de otra historia para poder implementarse y probarse. |
| Negociable | Sí | El detalle del QR o el formato exacto del código se puede ajustar sin cambiar el valor central. |
| Valiosa | Sí | Elimina la fila física de anotación en recepción. |
| Estimable | Sí | El equipo puede dimensionar el esfuerzo con la información disponible. |
| Pequeña | Sí | Cabe en una iteración corta. |
| Verificable | Sí | Se puede comprobar viendo el turno creado en Firestore y el código mostrado en pantalla. |

---

## HU-02a — Llamar turno

| Campo | Detalle |
|-------|---------|
| Historia | Como recepcionista, quiero llamar al siguiente turno en fila, para indicarle al paciente que debe dirigirse a la atención. |
| Módulo | Recepción |
| Requisitos relacionados | RF-05, RF-06 |

### Criterios de aceptación

1. Dado que hay turnos en estado "en-fila", cuando el recepcionista presiona "Llamar" sobre uno de ellos, entonces el turno cambia a "llamado", se registra `horaLlamado` y `llamadoPor`, y se agrega el evento al historial de llamadas.
2. Dado que dos recepcionistas llaman casi simultáneamente el mismo turno, cuando ambas acciones se procesan, entonces prevalece la última escritura en Firestore, sin aviso al operador cuya acción fue sobrescrita (limitación conocida, ver RNF-06).

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Sí | Depende de que existan turnos creados (HU-01), pero no requiere cambios en esa historia. |
| Negociable | Sí | El diseño de la tabla o del botón puede variar. |
| Valiosa | Sí | Es la operación central del día a día de recepción. |
| Estimable | Sí | |
| Pequeña | Sí | Una sola acción, un solo cambio de estado. |
| Verificable | Sí | Se verifica el cambio de estado y los campos `horaLlamado`/`llamadoPor` en Firestore. |

---

## HU-02b — Devolver turno

| Campo | Detalle |
|-------|---------|
| Historia | Como recepcionista, quiero devolver a la fila un turno que llamé por error, para corregir la equivocación sin que el paciente pierda su lugar. |
| Módulo | Recepción |
| Requisitos relacionados | RF-07 |

### Criterios de aceptación

1. Dado que un turno está en estado "llamado", cuando el recepcionista presiona "Devolver", entonces el turno vuelve a estado "en-fila" y vuelve a listarse en la pantalla de sala de espera.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Sí | |
| Negociable | Sí | |
| Valiosa | Sí | Permite corregir errores operativos sin perder el orden del paciente. |
| Estimable | Sí | |
| Pequeña | Sí | Una sola acción. |
| Verificable | Sí | Se verifica el cambio de estado en Firestore. |

---

## HU-02c — Finalizar turno

| Campo | Detalle |
|-------|---------|
| Historia | Como recepcionista, quiero finalizar un turno una vez atendido el paciente, para que deje de figurar como turno activo. |
| Módulo | Recepción |
| Requisitos relacionados | RF-08 |

### Criterios de aceptación

1. Dado que un turno fue atendido, cuando el recepcionista presiona "Finalizar", entonces el turno cambia a estado "finalizado" y deja de listarse como activo en el panel de recepción y en la pantalla de sala de espera.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Sí | |
| Negociable | Sí | |
| Valiosa | Sí | Mantiene el panel y la pantalla pública mostrando solo turnos realmente activos. |
| Estimable | Sí | |
| Pequeña | Sí | Una sola acción. |
| Verificable | Sí | Se verifica el cambio de estado en Firestore y su desaparición del panel/pantalla. |

---

## HU-03 — Visualización pública de la fila

| Campo | Detalle |
|-------|---------|
| Historia | Como paciente en sala de espera, quiero ver en una pantalla los turnos en fila y cuál está siendo llamado, para saber cuánto falta sin tener que preguntar. |
| Módulo | Pantalla de sala de espera |
| Requisitos relacionados | RF-11, RF-12 |

### Criterios de aceptación

1. Dado que hay turnos en estado "en-fila" o "llamado", cuando se carga la pantalla, entonces se listan en tiempo real.
2. Dado que un turno pasa a "llamado", cuando se actualiza la pantalla, entonces esa tarjeta se resalta visualmente respecto de las demás.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Sí | |
| Negociable | Sí | El estilo visual del resaltado es un detalle de implementación. |
| Valiosa | Sí | Reduce la incertidumbre y las consultas de los pacientes en sala. |
| Estimable | Sí | |
| Pequeña | Sí | |
| Verificable | Sí | Se verifica comparando el estado en Firestore contra lo mostrado en pantalla. |

---

## HU-04 — Seguimiento remoto del turno

| Campo | Detalle |
|-------|---------|
| Historia | Como paciente, quiero consultar el estado de mi turno desde mi celular con un link, para no depender de estar mirando la pantalla de la sala de espera todo el tiempo. |
| Módulo | Seguimiento |
| Requisitos relacionados | RF-13, RF-14, RF-15 |

### Criterios de aceptación

1. Dado que el paciente abre el link con su token, cuando el token es válido, entonces se muestra el estado actual de su turno.
2. Dado que el turno pasa a "llamado" mientras el paciente tiene la página abierta, entonces se reproduce una notificación sonora (si está habilitada).
3. Dado que el token ya fue consumido anteriormente, cuando se intenta acceder de nuevo con el mismo link, entonces el sistema bloquea el acceso y muestra un mensaje indicando que el enlace ya fue utilizado.
4. Dado que el token no existe o es inválido, cuando se accede a `seguimiento.html`, entonces el sistema muestra un mensaje de error claro, sin exponer datos de ningún turno.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Sí | |
| Negociable | Sí | |
| Valiosa | Sí | Le da autonomía al paciente para moverse dentro del predio sin perder su lugar. |
| Estimable | Sí | |
| Pequeña | Sí | |
| Verificable | Sí | Se verifica accediendo con un token real y observando el estado mostrado. |