# Casos de uso

## Diagrama general

Incluido en `diagramas/casos-de-uso.puml`. Visualizar en [plantuml.com](https://www.plantuml.com/plantuml/uml/).

**Actores identificados:**
- **Paciente** (actor principal): interactúa con `autorecepcion.html` para generar su turno y con `seguimiento.html` para consultarlo.
- **Recepcionista** (actor principal): interactúa con `recepcion.html` para gestionar el llamado de turnos.

No se identificaron relaciones `include`/`extend` entre los casos de uso relevados hasta el momento; se evaluará al avanzar el relevamiento si "Llamar turno" debería incluir un subcaso de notificación.

---

## CU-01 — Autogestionar turno

| Campo | Detalle |
|-------|---------|
| Identificador | CU-01 |
| Nombre | Autogestionar turno |
| Descripción | El paciente ingresa a la pantalla de autorecepción, completa sus datos y el sistema le asigna un turno con código visible y QR. |
| Actores | Principal: Paciente / Secundario: — |
| Precondiciones | El paciente accede a `autorecepcion.html` desde un navegador o dispositivo habilitado en el lugar. |
| Postcondiciones | Éxito: se crea un turno en Firestore (`turnos_fertya`) con código visible y token asignados. Fallo: no se crea el turno y no se muestra código. |

### Secuencia normal

| # | Acción (actor) | Reacción (sistema) |
|---|----------------|--------------------|
| 1 | El paciente accede a la pantalla de autorecepción. | Muestra el formulario de ingreso (DNI, motivo de visita). |
| 2 | El paciente completa los datos y envía el formulario. | Genera un token único, incrementa el contador de la categoría correspondiente y crea el turno en Firestore. |
| 3 | — | Muestra el código visible generado (ej. `C-001`) y un código QR asociado al token. |

### Excepciones

| # | Situación | Respuesta del sistema |
|---|-----------|-----------------------|
| E1 | El motivo ingresado no corresponde a ninguna categoría conocida. | Se asigna la categoría por defecto (`C`) y se continúa el flujo normal. |

| Campo | Detalle |
|-------|---------|
| Rendimiento | Respuesta esperada en pocos segundos (creación de documento + contador atómico en Firestore). |
| Frecuencia | Alta — Fertya recibe más de 70 pacientes por día entre tratamientos y controles, con la franja más crítica entre las 7 y las 11 hs (horario de laboratorio). |
| Importancia | Alta. |
| Urgencia | Media. |

---

## CU-02 — Llamar turno

| Campo | Detalle |
|-------|---------|
| Identificador | CU-02 |
| Nombre | Llamar turno |
| Descripción | El recepcionista selecciona el siguiente turno en fila y lo llama para atención. |
| Actores | Principal: Recepcionista / Secundario: — |
| Precondiciones | Debe existir al menos un turno en estado "en-fila"; el recepcionista debe estar operando el panel. |
| Postcondiciones | Éxito: el turno cambia a "llamado", con hora y responsable registrados. Fallo: el turno permanece en "en-fila". |

### Secuencia normal

| # | Acción (actor) | Reacción (sistema) |
|---|----------------|--------------------|
| 1 | El recepcionista abre el panel de turnos. | Muestra la tabla de turnos activos en tiempo real, ordenados por fecha de creación. |
| 2 | El recepcionista presiona "Llamar" sobre un turno. | Actualiza el estado a "llamado", registra `horaLlamado` y `llamadoPor`, y agrega el evento al historial de llamadas del turno. |

### Excepciones

| # | Situación | Respuesta del sistema |
|---|-----------|-----------------------|
| E1 | El recepcionista llama por error a un turno equivocado. | Puede usar la acción "Devolver" para volver el turno a estado "en-fila". |

| Campo | Detalle |
|-------|---------|
| Rendimiento | El cambio debe reflejarse en tiempo real en la pantalla de sala de espera. |
| Frecuencia | Alta — se repite por cada paciente atendido. |
| Importancia | Alta. |
| Urgencia | Alta. |
