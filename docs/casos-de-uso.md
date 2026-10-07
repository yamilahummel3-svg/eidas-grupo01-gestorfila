# Casos de uso

## Diagrama general

Incluido en `diagramas/casos-de-uso.puml`. Visualizar en [plantuml.com](https://www.plantuml.com/plantuml/uml/).

**Actores identificados:**
- **Paciente** (actor principal): genera su turno en `autorecepcion.html`, lo sigue desde el celular en `seguimiento.html` y mira la pantalla de sala de espera (`pantalla.html`).
- **Recepcionista** (actor principal): opera `recepcion.html` desde Planta Baja o el 4º piso (Recepción 1 a 4) para llamar, devolver y finalizar turnos, cerrar la jornada y exportar el reporte.
- **Firebase / Firestore** (actor secundario, sistema externo): guarda los turnos y los contadores, y avisa en tiempo real los cambios a todas las pantallas (RNF-01). Participa en todos los casos que leen o escriben turnos.

**Relaciones entre casos de uso:**

| Relación | Tipo | Justificación |
|----------|------|---------------|
| Autogestionar turno → Generar código visible y token | `include` | Siempre ocurre: no existe un turno sin código visible ni token de seguimiento (RF-02, RF-03). Se separa porque es la parte que incrementa el contador atómico por categoría. |
| Notificar llamado con sonido → Consultar seguimiento | `extend` | Solo ocurre bajo condición: el paciente habilitó el sonido y su turno pasa a "llamado" (RF-15). Sin sonido, el seguimiento funciona igual. |
| Devolver turno → Llamar turno | `extend` | Solo ocurre si el recepcionista llamó un turno por error o el paciente no se presentó (RF-07). No es parte del flujo normal del llamado. |

**Relación que se evaluó y se descartó:** "Cerrar jornada" `include` "Finalizar turno". Aunque las dos dejan turnos en estado "finalizado", no operan sobre lo mismo: cerrar jornada finaliza en bloque los turnos **"en-fila"** que nadie atendió, mientras que "Finalizar turno" opera sobre un turno **"llamado"**, de a uno y con confirmación propia. Un `include` diría que cerrar la jornada ejecuta "Finalizar turno", y no es así.

---

## CU-01 — Autogestionar turno

| Campo | Detalle |
|-------|---------|
| Identificador | CU-01 |
| Nombre | Autogestionar turno |
| Descripción | El paciente se registra al llegar ingresando su DNI y motivo de visita; el sistema le asigna un turno con código visible y un QR para el seguimiento. |
| Actores | Principal: Paciente / Secundario: Firebase / Firestore |
| Requisitos / HU | RF-01, RF-01b, RF-02, RF-03, RF-04, RNF-04 · HU-01 |
| Precondiciones | El paciente está en Fertya y accede a `autorecepcion.html` desde un dispositivo habilitado en el lugar o desde su celular. |
| Postcondiciones | Éxito: se crea un turno en `turnos_fertya` con estado "en-fila", código visible, token y hora de ingreso; el contador de la categoría queda incrementado en 1. Fallo: no se crea el turno y el contador no cambia (la creación y el incremento se hacen en una misma transacción). |

### Secuencia normal

| # | Acción (actor) | Reacción (sistema) |
|---|----------------|--------------------|
| 1 | El paciente accede a la pantalla de autorecepción. | Muestra el formulario con dos campos: DNI y motivo de la visita (lista desplegable). |
| 2 | Ingresa su DNI, elige el motivo y presiona "Registrar llegada". | Valida que el DNI tenga 7 u 8 dígitos numéricos y que haya un motivo elegido (RF-01b). |
| 3 | — | **Incluye "Generar código visible y token":** genera un token único, incrementa el contador de la categoría del motivo y crea el turno en estado "en-fila", todo en una misma transacción (RF-02, RF-03). |
| 4 | — | Muestra el código visible (ej. `C-001`) en tamaño grande y un QR con el enlace de seguimiento; limpia el formulario para el próximo paciente (RF-04). |
| 5 | El paciente escanea el QR con su celular (opcional). | Oculta el QR en la pantalla de autorecepción y muestra "Código escaneado. Gracias.", para que otro paciente no lo reutilice. |

### Excepciones

| # | Situación | Respuesta del sistema |
|---|-----------|-----------------------|
| E1 | El DNI tiene letras, puntos o menos de 7 / más de 8 dígitos (paso 2). | Muestra "Ingresá un DNI válido (7–8 dígitos)." y no crea el turno. El paciente corrige y vuelve al paso 2. |
| E2 | El paciente no eligió un motivo (paso 2). | Muestra "Seleccioná un motivo." y no crea el turno. |
| E3 | Falla la conexión con Firestore o la transacción no se completa (paso 3). | Muestra "Error registrando. Avisá en recepción." No se crea el turno ni se incrementa el contador; el paciente queda a cargo de recepción. |
| E4 | Dos pacientes se registran en el mismo instante con la misma categoría (paso 3). | La transacción garantiza que cada uno reciba un número distinto (ej. `C-014` y `C-015`); nunca se repite el código. |
| E5 | El paciente se registra dos veces con el mismo DNI (por ejemplo, porque creyó que no había funcionado). | El sistema no lo impide: se crean dos turnos. El recepcionista finaliza el duplicado desde el panel. Se documenta como mejora pendiente (validar DNI repetido en el día). |
| E6 | El paciente se va sin escanear el QR (paso 5). | El turno igual queda en fila y se ve en la pantalla de sala. El QR no se puede recuperar después, así que no tendrá seguimiento desde el celular. |

| Campo | Detalle |
|-------|---------|
| Rendimiento | Menos de 3 segundos desde "Registrar llegada" hasta ver el código y el QR (RNF-08). El paciente completa el registro en menos de 30 segundos sin ayuda (RNF-04). |
| Frecuencia | Más de 70 veces por día (una por paciente). Estimado: unas 40 en la franja crítica de 7 a 11 hs, es decir, alrededor de 10 por hora. |
| Importancia | Alta: es la puerta de entrada al sistema; sin turno no hay nada que gestionar. |
| Urgencia | Alta: reemplaza el circuito duplicado PB → 4º piso relevado en las entrevistas. |

---

## CU-02 — Llamar turno

| Campo | Detalle |
|-------|---------|
| Identificador | CU-02 |
| Nombre | Llamar turno |
| Descripción | El recepcionista llama a un paciente en fila para que se acerque a su mostrador. |
| Actores | Principal: Recepcionista / Secundario: Firebase / Firestore |
| Requisitos / HU | RF-05, RF-06, RNF-01, RNF-06 · HU-02a |
| Precondiciones | Hay al menos un turno del día en estado "en-fila". El recepcionista tiene abierto `recepcion.html` y eligió su puesto (Recepción 1 a 4). |
| Postcondiciones | Éxito: el turno queda en estado "llamado", con `horaLlamado` y `llamadoPor` registrados y el llamado agregado al historial; la pantalla de sala y el seguimiento del paciente lo muestran como llamado. Fallo: el turno sigue en "en-fila". |

### Secuencia normal

| # | Acción (actor) | Reacción (sistema) |
|---|----------------|--------------------|
| 1 | El recepcionista abre el panel y elige su puesto en el selector "Recepción". | Muestra la tabla de turnos del día en tiempo real, ordenados por hora de ingreso; recuerda el puesto elegido para las próximas veces (RF-05). |
| 2 | Presiona "Llamar" sobre un turno en fila. | Cambia el estado a "llamado", registra la hora y el puesto que llamó, y agrega el llamado al historial del turno (RF-06). |
| 3 | — | En la fila del turno reemplaza "Llamar" por "Devolver" y "Finalizar". |
| 4 | — | La pantalla de sala resalta la tarjeta con "LLAMANDO" y el puesto; el celular del paciente muestra el banner de llamado (RNF-01). |

### Excepciones

| # | Situación | Respuesta del sistema |
|---|-----------|-----------------------|
| E1 | Recepción PB y Recepción 4º piso llaman el mismo turno casi al mismo tiempo. | Prevalece la última escritura: `llamadoPor` y `horaLlamado` muestran solo el último llamado y el operador que llamó primero no recibe aviso (limitación conocida LIM-01; incumple RNF-06). |
| E2 | La transacción del llamado falla (paso 2). | El sistema reintenta con una actualización simple: el turno queda "llamado" con hora y puesto, pero **ese llamado no se agrega al historial** (ver nota). |
| E3 | Entre que el panel muestra el turno "en-fila" y el recepcionista presiona "Llamar", otro puesto lo finalizó (paso 2). | El sistema no verifica el estado actual antes de llamar: el turno finalizado vuelve a quedar "llamado" y reaparece en la pantalla de sala. El recepcionista debe finalizarlo de nuevo. Mejora propuesta: verificar el estado dentro de la transacción (la misma que resolvería LIM-01). |

**Flujos alternativos (decisiones del recepcionista, no fallas del sistema):**

- **Llamó el turno equivocado** (después del paso 2): usa "Devolver turno", que extiende este caso de uso. El turno vuelve a "en-fila" sin perder su lugar por hora de ingreso (RF-07).
- **El paciente llamado no se presenta al mostrador:** el recepcionista lo devuelve a la fila para llamarlo más tarde o finaliza el turno.

> **Nota técnica a verificar:** el código arma el historial (`llamadas`) con una marca de hora del servidor dentro de un arreglo, algo que Firestore no admite. Si es así, la transacción falla siempre y E2 es en realidad el camino habitual: el historial de llamados no se está guardando. Verificarlo en la consola de Firestore mirando si los turnos llamados tienen el campo `llamadas`.

| Campo | Detalle |
|-------|---------|
| Rendimiento | El cambio de estado se ve en la pantalla de sala y en el celular del paciente en menos de 2 segundos (RNF-01). |
| Frecuencia | Al menos una vez por paciente: más de 70 por día, concentradas entre las 7 y las 11 hs. |
| Importancia | Alta: es la operación central del día a día de recepción. |
| Urgencia | Alta. |

---

## CU-03 — Consultar seguimiento de turno

| Campo | Detalle |
|-------|---------|
| Identificador | CU-03 |
| Nombre | Consultar seguimiento de turno |
| Descripción | El paciente sigue el estado de su turno desde su celular, a través del enlace del QR, sin tener que mirar la pantalla de la sala. |
| Actores | Principal: Paciente / Secundario: Firebase / Firestore |
| Requisitos / HU | RF-13, RF-14, RF-15, RNF-01, RNF-05 · HU-04 |
| Precondiciones | El paciente generó un turno (CU-01) y escaneó el QR, que abre `seguimiento.html` con su token en la URL. |
| Postcondiciones | Éxito: el paciente ve el estado actualizado de su turno; en la primera apertura el token queda marcado como consumido, con fecha, hora y dispositivo. Fallo: se muestra un mensaje de error sin datos de ningún turno. |

### Secuencia normal

| # | Acción (actor) | Reacción (sistema) |
|---|----------------|--------------------|
| 1 | El paciente escanea el QR y abre el enlace. | Busca el turno asociado al token (RF-13). |
| 2 | — | Si es la primera vez que se abre, marca el token como consumido y registra fecha, hora y dispositivo (RF-14). |
| 3 | — | Muestra "Turno confirmado: C-002" y "Estás en fila. Esperá a ser llamado." |
| 4 | (Opcional) Marca "Reproducir sonido al ser llamado". | Guarda la preferencia en el celular. |
| 5 | Espera con la página abierta. | Cuando el turno pasa a "llamado", muestra un banner con el código y el puesto que lo llama (RNF-01). **Extiende a "Notificar llamado con sonido"** si el paciente habilitó el sonido (RF-15). |
| 6 | — | Cuando el turno se finaliza, muestra "Tu turno ya fue finalizado." |

### Excepciones

| # | Situación | Respuesta del sistema |
|---|-----------|-----------------------|
| E1 | El enlace se abre sin token (por ejemplo, alguien entra directo a `seguimiento.html`). | Muestra "Token ausente en la URL." y no muestra datos. |
| E2 | El token no existe o está mal copiado. | Muestra "Código inválido o no encontrado." sin exponer datos de ningún turno. |
| E3 | Falla la conexión al abrir el enlace. | Muestra "Error procesando el código. Intentá nuevamente o avisá en recepción." |
| E4 | El paciente reabre el enlace o lo comparte con un acompañante. | El seguimiento **sigue funcionando**: el token se marca como consumido solo la primera vez, pero no se bloquea (RF-14). Quien tenga el enlace ve el código y el estado del turno, sin datos personales (limitación conocida LIM-05). |
| E5 | El navegador del celular bloquea el audio. | No suena, pero el banner visual de llamado se muestra igual (paso 5). |

> **Nota de coherencia:** al consumirse el token, el turno deja de mostrarse en la pantalla de sala mientras está "en-fila" (vuelve a verse cuando lo llaman). Es decir, quien escanea el QR sigue su turno desde el celular y no en la pantalla. Este comportamiento está documentado en RF-11.

| Campo | Detalle |
|-------|---------|
| Rendimiento | El paso a "llamado" se ve en el celular en menos de 2 segundos (RNF-01). La página abre en menos de 3 segundos con conexión 4G. |
| Frecuencia | Una apertura por cada paciente que escanea el QR; luego la página queda abierta y se actualiza sola hasta que lo llaman. |
| Importancia | Media: el paciente puede usar la pantalla de sala, pero el seguimiento le permite moverse por el edificio sin perder el llamado. |
| Urgencia | Media. |

---

## CU-04 — Cerrar jornada

| Campo | Detalle |
|-------|---------|
| Identificador | CU-04 |
| Nombre | Cerrar jornada |
| Descripción | Al final del día, el recepcionista finaliza en bloque todos los turnos que quedaron en fila, o solo limpia la vista de su panel. |
| Actores | Principal: Recepcionista / Secundario: Firebase / Firestore |
| Requisitos / HU | RF-09, RF-09b · HU-05 |
| Precondiciones | Terminó la atención del día. El recepcionista tiene abierto `recepcion.html`. |
| Postcondiciones | Éxito (cerrar): todos los turnos del día que estaban "en-fila" quedan "finalizado" con `horaFinalizado`. Éxito (limpiar): el panel local queda vacío y la base no cambia. Fallo: ver E1. |

### Secuencia normal

| # | Acción (actor) | Reacción (sistema) |
|---|----------------|--------------------|
| 1 | Presiona "Cerrar jornada / Limpiar panel". | Muestra una ventana de confirmación: Aceptar finaliza los turnos en la base; Cancelar solo limpia la vista. |
| 2 | Presiona "Aceptar". | Cambia a "finalizado", con `horaFinalizado`, todos los turnos del día en estado "en-fila", en lotes de hasta 400 (RF-09). |
| 3 | — | Muestra "Jornada finalizada ✔". |

**Flujo alternativo (paso 2):** si presiona "Cancelar", el sistema vacía solo la vista de su panel, sin tocar la base (RF-09b), y cambia el botón a "Mostrar jornada" para restaurarla.

### Excepciones

| # | Situación | Respuesta del sistema |
|---|-----------|-----------------------|
| E1 | Falla la conexión en medio del cierre (paso 2). | Muestra "Ocurrió un error al finalizar la jornada." Los lotes que ya se procesaron quedan finalizados y el resto sigue en fila: **cierre parcial**. El recepcionista puede repetir la acción, que solo toma los turnos que siguen "en-fila". |
| E2 | Quedan turnos en estado "llamado" al cerrar. | No se finalizan (el cierre solo toma los "en-fila"); el recepcionista debe finalizarlos uno por uno. |
| E3 | El recepcionista presiona "Aceptar" por error con pacientes esperando. | Todos los turnos en fila pasan a "finalizado" y el panel no permite deshacerlo. Los pacientes deben registrarse de nuevo (CU-01). Se documenta como riesgo: la confirmación es la única protección. |

| Campo | Detalle |
|-------|---------|
| Rendimiento | Menos de 5 segundos para cerrar una jornada de 100 turnos. |
| Frecuencia | Una vez por día, al cierre. |
| Importancia | Media: deja la base ordenada para el día siguiente y para el reporte CSV. |
| Urgencia | Baja. |
