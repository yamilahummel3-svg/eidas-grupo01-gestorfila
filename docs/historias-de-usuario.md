# Historias de usuario

_Presentar al menos una historia de usuario representativa por módulo._
_Cada historia debe incluir formato clásico, criterios de aceptación y validación INVEST._

---

## HU-01 — Autogestión de turno

| Campo | Detalle |
|-------|---------|
| Historia | Como paciente, quiero registrarme en la fila ingresando mi DNI y motivo de visita, para obtener un turno sin necesidad de hacer fila física en recepción. |
| Módulo | Autorecepción |
| Requisitos relacionados | RF-01, RF-01b, RF-02, RF-03, RF-04, RNF-04, RNF-08 |

### Criterios de aceptación

1. Dado que el paciente completa DNI y motivo de visita, cuando envía el formulario, entonces el sistema genera un código visible (ej. `C-001`) y lo muestra en pantalla junto a un código QR en menos de 3 segundos (RNF-08).
2. Dado que el paciente elige el motivo de la lista, cuando se genera el turno, entonces la letra del código corresponde a la categoría del motivo (ej. Laboratorio → `L-`).
3. Dado que se genera el turno, cuando se crea en Firestore, entonces se le asocia un token único para el seguimiento posterior.
4. Dado que el DNI ingresado no es numérico o no tiene entre 7 y 8 dígitos, o que no se eligió un motivo, cuando el paciente intenta enviar el formulario, entonces el sistema rechaza el envío y muestra un aviso, sin crear el turno.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Sí | Es la primera historia del circuito: no necesita que exista ninguna otra para implementarse y probarse. Las demás dependen de ella, no al revés. |
| Negociable | Sí | El tamaño del QR, el texto de confirmación o el formato exacto del código se pueden ajustar con Fertya sin cambiar el valor central: registrarse sin hacer fila. |
| Valiosa | Sí | Elimina el circuito duplicado relevado en las entrevistas (primer contacto en PB y segunda fila en el 4º piso): el paciente queda registrado una sola vez. |
| Estimable | Sí | Son dos campos, una validación y una transacción en Firestore; el equipo ya conocía cada pieza, por eso se pudo dimensionar sin dudas. |
| Pequeña | Sí | Cubre una sola pantalla y una sola acción del paciente; entra en una iteración corta. |
| Verificable | Sí | Se prueba registrando un DNI válido y comprobando que aparece el código con el QR en menos de 3 segundos (RNF-08), y con un DNI de 6 dígitos comprobando que aparece el aviso y no se crea el turno en Firestore. |

---

## HU-02a — Llamar turno

| Campo | Detalle |
|-------|---------|
| Historia | Como recepcionista, quiero llamar al siguiente turno en fila, para indicarle al paciente que debe dirigirse a mi mostrador. |
| Módulo | Recepción |
| Requisitos relacionados | RF-05, RF-06, RNF-01, RNF-06 |

### Criterios de aceptación

1. Dado que hay turnos en estado "en-fila", cuando el recepcionista presiona "Llamar" sobre uno de ellos, entonces el turno cambia a "llamado" y se registran `horaLlamado` y `llamadoPor` (el puesto elegido, Recepción 1 a 4).
2. Dado que el turno pasó a "llamado", cuando se consulta la pantalla de sala de espera, entonces la tarjeta aparece resaltada con "LLAMANDO" en menos de 2 segundos (RNF-01).
3. Dado que dos recepcionistas llaman casi simultáneamente el mismo turno, cuando ambas acciones se procesan, entonces prevalece la última escritura en Firestore, sin aviso al operador cuya acción fue sobrescrita (limitación conocida LIM-01; incumple RNF-06).

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Parcial | Necesita que existan turnos creados (HU-01) para poder probarse, pero no requiere cambios en esa historia: se puede desarrollar en paralelo cargando turnos de prueba. |
| Negociable | Sí | El diseño de la tabla, la ubicación del botón o si se llama "el siguiente" automáticamente se puede discutir con las coordinadoras de recepción. |
| Valiosa | Sí | Es la operación central del día a día de recepción y la que reemplaza el llamado verbal en el pasillo del 4º piso. |
| Estimable | Sí | Es un único cambio de estado sobre un documento existente, más el registro de hora y puesto; el esfuerzo es acotado y conocido. |
| Pequeña | Sí | Una sola acción y un solo cambio de estado. Por eso se separó de devolver y finalizar (antes era HU-02). |
| Verificable | Sí | Se verifica en Firestore que el estado pasa a "llamado" con `horaLlamado` y `llamadoPor`, y se cronometra que la pantalla de sala lo muestre en menos de 2 segundos (RNF-01). |

---

## HU-02b — Devolver turno

| Campo | Detalle |
|-------|---------|
| Historia | Como recepcionista, quiero devolver a la fila un turno que llamé por error o cuyo paciente no se presentó, para corregir la situación sin que el paciente pierda su lugar. |
| Módulo | Recepción |
| Requisitos relacionados | RF-07 |

### Criterios de aceptación

1. Dado que un turno está en estado "llamado", cuando el recepcionista presiona "Devolver", entonces el turno vuelve a estado "en-fila" y se borra su hora de llamado.
2. Dado que el turno fue devuelto, cuando se consulta el panel, entonces conserva su posición según la hora de ingreso original y vuelve a mostrar el botón "Llamar".

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Parcial | Solo tiene sentido sobre un turno llamado (HU-02a), pero se implementa como una acción separada que no modifica la lógica del llamado. |
| Negociable | Sí | Se puede discutir si devolver pide confirmación o si registra un motivo (error vs. paciente ausente). |
| Valiosa | Sí | Permite corregir errores operativos sin que el paciente pierda su lugar, algo que con la fila física no se podía garantizar. |
| Estimable | Sí | Es la operación inversa al llamado; el equipo la estimó en base a HU-02a. |
| Pequeña | Sí | Un botón y un cambio de estado. |
| Verificable | Sí | Se verifica que el estado vuelve a "en-fila", que `horaLlamado` queda vacío y que el turno mantiene su orden por hora de ingreso en el panel. |

---

## HU-02c — Finalizar turno

| Campo | Detalle |
|-------|---------|
| Historia | Como recepcionista, quiero finalizar un turno una vez atendido el paciente, para que deje de figurar como pendiente de atención. |
| Módulo | Recepción |
| Requisitos relacionados | RF-08 |

### Criterios de aceptación

1. Dado que un turno está en estado "llamado", cuando el recepcionista presiona "Finalizar" y confirma en la ventana de confirmación, entonces el turno cambia a "finalizado" y se registra `horaFinalizado`.
2. Dado que el turno fue finalizado, cuando se consulta el panel, entonces sigue listado en la tabla del día con estado "finalizado" y sin botones de acción; y deja de mostrarse en la pantalla de sala de espera.
3. Dado que el recepcionista presiona "Finalizar", cuando elige "Cancelar" en la confirmación, entonces el turno sigue en "llamado" sin cambios.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Parcial | Solo se habilita sobre turnos llamados (HU-02a), pero es una acción separada que se puede desarrollar y probar por su cuenta. |
| Negociable | Sí | Se puede discutir si los finalizados deben ocultarse del panel (hoy siguen listados para tener el historial del día a la vista). |
| Valiosa | Sí | La hora de finalización es el dato que permite medir cuánto duró cada atención, un problema relevado en las entrevistas (no había registro de tiempos). |
| Estimable | Sí | Es un cambio de estado más una confirmación; mismo esfuerzo que HU-02b. |
| Pequeña | Sí | Un botón, una confirmación y un cambio de estado. |
| Verificable | Sí | Se verifica en Firestore el estado "finalizado" con `horaFinalizado`, y que la tarjeta desaparece de la pantalla de sala. |

---

## HU-03 — Visualización pública de la fila

| Campo | Detalle |
|-------|---------|
| Historia | Como paciente en sala de espera, quiero ver en una pantalla los turnos en fila y cuál está siendo llamado, para saber cuándo me toca sin tener que preguntar. |
| Módulo | Pantalla de sala de espera |
| Requisitos relacionados | RF-11, RF-12, RNF-01, RNF-03, RNF-09 |

### Criterios de aceptación

1. Dado que hay turnos en estado "en-fila" o "llamado", cuando se carga la pantalla, entonces se listan en tiempo real, sin nombres ni DNI, solo con código y motivo.
2. Dado que un turno pasa a "llamado", cuando se actualiza la pantalla, entonces esa tarjeta se resalta con la etiqueta "LLAMANDO" y el texto "Llamado por: Recepción N" en menos de 2 segundos (RNF-01).

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Parcial | Muestra datos que generan HU-01 y HU-02a, pero solo los lee: se puede desarrollar con turnos de prueba sin tocar esas historias. |
| Negociable | Sí | El estilo del resaltado, el tamaño de las tarjetas y la cantidad de turnos visibles se pueden ajustar según el monitor real de la sala. |
| Valiosa | Sí | Reduce las consultas de los pacientes al mostrador y la incertidumbre en una espera que, en medicina reproductiva, ya tiene carga emocional. |
| Estimable | Sí | Es una pantalla de solo lectura con un listener en tiempo real; el equipo ya había resuelto ese mecanismo en el panel de recepción. |
| Pequeña | Sí | Una sola pantalla sin interacción del usuario. |
| Verificable | Sí | Se llama un turno desde recepción y se cronometra que la tarjeta aparezca resaltada en menos de 2 segundos (RNF-01); se comprueba que no aparece ningún DNI (RNF-03). |

---

## HU-04 — Seguimiento remoto del turno

| Campo | Detalle |
|-------|---------|
| Historia | Como paciente, quiero consultar el estado de mi turno desde mi celular con un link, para no depender de estar mirando la pantalla de la sala de espera todo el tiempo. |
| Módulo | Seguimiento |
| Requisitos relacionados | RF-13, RF-14, RF-15, RNF-01, RNF-05 |

### Criterios de aceptación

1. Dado que el paciente abre el link con su token, cuando el token es válido, entonces se muestra su código y el estado actual de su turno.
2. Dado que el turno pasa a "llamado" mientras el paciente tiene la página abierta, entonces se muestra un banner con el código y el puesto que lo llama en menos de 2 segundos, y se reproduce un sonido si el paciente lo habilitó.
3. Dado que el paciente abre el link por primera vez, cuando se carga la página, entonces el token queda marcado como consumido con fecha y hora; y si vuelve a abrir el mismo link, puede seguir viendo el estado de su turno.
4. Dado que el token no existe o falta en la URL, cuando se accede a `seguimiento.html`, entonces el sistema muestra un mensaje de error claro, sin exponer datos de ningún turno.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Parcial | Necesita el token que genera HU-01, pero no modifica esa historia; se puede probar con un token de un turno de prueba. |
| Negociable | Sí | Se puede discutir con Fertya si el enlace debería vencer al finalizar el turno (hoy no vence, ver LIM-05) o si el sonido debería venir activado. |
| Valiosa | Sí | Le da autonomía al paciente para moverse dentro del edificio (por ejemplo, entre PB y el 4º piso) sin perder su llamado. |
| Estimable | Sí | Reutiliza el listener en tiempo real de las otras pantallas; lo nuevo es la búsqueda por token y el aviso sonoro. |
| Pequeña | Sí | Una sola pantalla con tres estados (en fila, llamado, finalizado) más el mensaje de error. |
| Verificable | Sí | Se abre el link de un turno de prueba, se llama el turno desde recepción y se comprueba el banner en menos de 2 segundos; se abre el link con un token inventado y se comprueba el mensaje de error. |

---

## HU-05 — Cerrar jornada

| Campo | Detalle |
|-------|---------|
| Historia | Como recepcionista, quiero cerrar la jornada finalizando de una vez todos los turnos que quedaron en fila, para empezar el día siguiente con el panel limpio y el reporte del día completo. |
| Módulo | Recepción |
| Requisitos relacionados | RF-09, RF-09b |

### Criterios de aceptación

1. Dado que quedan turnos del día en estado "en-fila", cuando el recepcionista presiona "Cerrar jornada / Limpiar panel" y elige "Aceptar", entonces todos esos turnos pasan a "finalizado" con `horaFinalizado` y se muestra "Jornada finalizada ✔".
2. Dado que el recepcionista elige "Cancelar" en la confirmación, cuando se cierra la ventana, entonces solo se vacía la vista de su panel, los turnos en Firestore no cambian y el botón pasa a "Mostrar jornada" para restaurar la vista.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Parcial | Reutiliza la operación de finalizar (HU-02c), pero aplicada en bloque; se puede probar por separado con turnos de prueba en fila. |
| Negociable | Sí | Se puede discutir si el cierre también debería reiniciar el contador de códigos (hoy es manual y mensual, ver LIM-02). |
| Valiosa | Sí | Evita que los turnos sin atender queden abiertos indefinidamente y ensucien el panel y el reporte del día siguiente. |
| Estimable | Sí | Es una consulta de los turnos en fila y una actualización en lote; el esfuerzo es acotado. |
| Pequeña | Sí | Un botón con dos caminos (cerrar o limpiar la vista). |
| Verificable | Sí | Se crean tres turnos de prueba, se cierra la jornada y se comprueba en Firestore que los tres quedaron "finalizado"; se repite con "Cancelar" y se comprueba que no cambian. |

---

## HU-06 — Exportar reporte del día

| Campo | Detalle |
|-------|---------|
| Historia | Como coordinadora de recepción, quiero descargar un reporte CSV con los turnos del día, para analizar los tiempos de espera y de atención sin depender de registros manuales. |
| Módulo | Recepción |
| Requisitos relacionados | RF-10 |

### Criterios de aceptación

1. Dado que hay turnos del día, cuando se presiona "Exportar día (CSV)", entonces se descarga el archivo `turnos_fertya.csv` con las columnas DNI, código, motivo, hora de ingreso, hora de llamado, hora de finalización y estado.
2. Dado que el archivo se descargó, cuando se abre en una planilla de cálculo, entonces hay una fila por cada turno del día, en el mismo orden que el panel (por hora de ingreso).

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Sí | Solo lee los turnos del día; no depende de que otra historia esté terminada para desarrollarse (puede probarse con turnos cargados a mano). |
| Negociable | Sí | Las columnas del reporte o el formato (CSV vs. planilla) se pueden acordar con la gerencia. |
| Valiosa | Sí | Responde al problema relevado de no tener "ningún registro objetivo" de cuánto esperaba cada paciente: con ingreso, llamado y finalización se calcula la espera y la duración de la atención. |
| Estimable | Sí | Es una consulta del día y la generación de un archivo de texto; el esfuerzo es chico y conocido. |
| Pequeña | Sí | Un botón y un archivo. |
| Verificable | Sí | Se descarga el archivo y se comprueba que tiene las 7 columnas y la misma cantidad de filas que turnos del día muestra el panel. |

---

## HU-07 — Aviso de paciente nuevo con el panel en segundo plano

| Campo | Detalle |
|-------|---------|
| Historia | Como recepcionista, quiero que el sistema me avise cuando llega un paciente nuevo aunque esté usando otra pestaña o programa, para no dejarlo esperando por no estar mirando el panel. |
| Módulo | Recepción |
| Requisitos relacionados | RF-16 |

### Criterios de aceptación

1. Dado que la pestaña del panel no está visible y el recepcionista autorizó las notificaciones del navegador, cuando un paciente se registra, entonces aparece una notificación "Hay N pacientes en espera" y el título de la pestaña empieza a titilar.
2. Dado que el título está titilando, cuando el recepcionista vuelve a la pestaña del panel, entonces el título deja de titilar.
3. Dado que el recepcionista no autorizó las notificaciones del navegador, cuando un paciente se registra con la pestaña oculta, entonces igual titila el título de la pestaña.

### Validación INVEST

| Criterio | ¿Se cumple? | Observación |
|----------|-------------|-------------|
| Independiente | Parcial | Necesita turnos que se creen (HU-01) para dispararse, pero se implementa sobre el listener que ya tiene el panel, sin cambiar otras historias. |
| Negociable | Sí | Se puede discutir con las coordinadoras si el aviso también debería sonar, o si solo debe avisar en la franja crítica de 7 a 11 hs. |
| Valiosa | Sí | Las recepcionistas atienden teléfono y otras tareas en la misma computadora; sin el aviso, un paciente nuevo podría esperar sin que nadie lo vea en el panel. |
| Estimable | Sí | Usa la API de notificaciones del navegador y el evento de visibilidad de la pestaña, dos piezas conocidas y acotadas. |
| Pequeña | Sí | Un solo aviso con dos variantes (notificación y título). |
| Verificable | Sí | Con el panel en una pestaña oculta, se registra un turno desde autorecepción y se comprueba que llega la notificación y titila el título; al volver a la pestaña, se comprueba que deja de titilar. |
