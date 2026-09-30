# Requisitos del sistema

## Descripción del sistema

Gestor de Filas Fertya es un sistema web para la gestión digital de la fila de pacientes de Fertya (medicina reproductiva). Reemplaza el llamado manual/verbal de pacientes por un circuito con autogestión de turno, panel de recepción en tiempo real, pantalla pública de sala de espera y seguimiento remoto vía token. Está desplegado sobre Firebase (Hosting + Firestore), con autenticación anónima y despliegue automático desde GitHub Actions.

## Requisitos funcionales

### Módulo 1 — Autorecepción de pacientes

| ID | Requisito |
|----|-----------|
| RF-01 | El sistema debe permitir al paciente ingresar su DNI y elegir el motivo de visita de una lista cerrada (Consultas, Laboratorio, Espermograma, Análisis / Ecografías, Otras gestiones) para generar un turno. |
| RF-01b | El sistema debe validar que el DNI ingresado sea numérico y tenga entre 7 y 8 dígitos, y que se haya elegido un motivo, rechazando el envío del formulario con un mensaje si no se cumple. |
| RF-02 | El sistema debe generar un token único (UUID) asociado a cada turno creado. |
| RF-03 | El sistema debe asignar un código visible secuencial según la categoría del motivo (ej. `C-001`, `L-002`), incrementando un contador atómico por categoría. El contador **no se reinicia automáticamente**: el equipo de desarrollo lo reinicia en forma **manual una vez por mes**, al inicio del mes y antes de la apertura, directamente sobre la colección `counters` de Firestore. |
| RF-04 | El sistema debe mostrar al paciente el código visible generado junto a un código QR con el enlace de seguimiento. Cuando el QR es escaneado, el sistema debe ocultarlo de la pantalla de autorecepción para que no lo reutilice otro paciente. |

### Módulo 2 — Recepción (gestión de turnos)

| ID | Requisito |
|----|-----------|
| RF-05 | El sistema debe mostrar al personal de recepción todos los turnos del día (en fila, llamados y finalizados) en tiempo real, ordenados por hora de ingreso, con código, DNI, motivo, hora de ingreso, hora de llamado, puesto que llamó y estado. |
| RF-06 | El sistema debe permitir cambiar el estado de un turno de "en-fila" a "llamado", registrando la hora y el puesto de recepción (Recepción 1 a 4) que realizó el llamado. |
| RF-07 | El sistema debe permitir devolver un turno de "llamado" a "en-fila". |
| RF-08 | El sistema debe permitir finalizar individualmente un turno en estado "llamado", previa confirmación, registrando la hora de finalización. |
| RF-09 | El sistema debe permitir, al cerrar la jornada, finalizar de forma masiva todos los turnos del día que quedaron en estado "en-fila", previa confirmación. |
| RF-09b | El sistema debe permitir "limpiar" la vista local del panel de recepción sin modificar el estado de los turnos en Firestore — es una acción puramente visual, distinta y no equivalente a "Cerrar jornada" (RF-09). |
| RF-10 | El sistema debe permitir exportar un reporte de los turnos del día en formato CSV, con las columnas DNI, código, motivo, hora de ingreso, hora de llamado, hora de finalización y estado. |
| RF-16 | El sistema debe avisar al recepcionista cuando ingresa un paciente nuevo mientras la pestaña del panel no está visible, mediante una notificación del navegador y el título de la pestaña intermitente. |

### Módulo 3 — Pantalla de sala de espera

| ID | Requisito |
|----|-----------|
| RF-11 | El sistema debe mostrar en tiempo real los turnos en estado "llamado" y los turnos "en-fila" cuyo paciente no está usando el seguimiento remoto (token no consumido). Quien escaneó el QR sigue su turno desde el celular y vuelve a aparecer en la pantalla cuando lo llaman. |
| RF-12 | El sistema debe resaltar visualmente el/los turno(s) en estado "llamado", indicando con texto el puesto de recepción que llama. |

### Módulo 4 — Seguimiento del paciente

| ID | Requisito |
|----|-----------|
| RF-13 | El sistema debe permitir a un paciente consultar el estado de su propio turno accediendo mediante un token único incluido en la URL, y mostrar un mensaje de error sin datos de ningún turno si el token falta o no existe. |
| RF-14 | El sistema debe marcar el token como consumido la primera vez que se abre el enlace, registrando fecha/hora y dispositivo de consumo. El token consumido **no bloquea** el seguimiento: el paciente puede reabrir el enlace y seguir viendo el estado de su turno. |
| RF-15 | El sistema debe notificar al paciente cuando su turno pasa a estado "llamado", con un banner visual siempre y con un sonido si el paciente lo habilitó. |

## Requisitos no funcionales

### Rendimiento y disponibilidad

| ID | Requisito |
|----|-----------|
| RNF-01 | Los cambios de estado de un turno deben verse en el panel de recepción, en la pantalla de sala de espera y en el seguimiento del paciente en **menos de 2 segundos**, sin recargar la página (listeners de Firestore). |
| RNF-02 | El sistema debe estar disponible públicamente vía Firebase Hosting, con una disponibilidad objetivo del **99%** en el horario de atención de Fertya (7 a 20 hs), y con despliegue automático ante cada cambio integrado a la rama principal del repositorio. |
| RNF-08 | La creación de un turno (desde "Registrar llegada" hasta ver el código y el QR) debe completarse en **menos de 3 segundos**. |

### Seguridad y privacidad

| ID | Requisito |
|----|-----------|
| RNF-03 | El sistema no debe mostrar nombres de pacientes en ninguna pantalla pública; en la sala de espera y en el seguimiento solo se muestra el código visible. El acceso a Firestore se realiza mediante autenticación anónima y debe acotarse con reglas de seguridad. **Limitaciones conocidas de la versión actual:** (a) el panel de recepción no requiere inicio de sesión: cualquiera con la URL puede operar turnos y ver los DNI; (b) la pantalla de sala de espera lee la colección de turnos sin autenticarse, lo que indica que las reglas permiten leer los turnos (incluido el DNI) sin sesión. Al almacenarse datos personales de pacientes en la nube, queda pendiente confirmar con el área de sistemas/legal de Grupo Oroño el cumplimiento de la Ley 25.326 de Protección de Datos Personales antes de una puesta en producción total. |
| RNF-05 | El token de seguimiento debe ser un UUID generado aleatoriamente (122 bits aleatorios), de modo que no pueda adivinarse ni deducirse a partir del código visible. **Limitación conocida:** el enlace no expira ni se bloquea después del primer uso; si se comparte, otra persona puede ver el estado de ese turno (solo código y estado, sin datos personales). Mejora propuesta: invalidar el enlace al finalizar el turno. |
| RNF-06 | El sistema puede presentar condiciones de carrera cuando dos operadores de recepción (por ejemplo, de Planta Baja y del 4º piso) intentan llamar el mismo turno de forma casi simultánea; en ese caso, prevalece la última escritura en Firestore, sin aviso al operador cuya acción fue sobrescrita. Se documenta como limitación conocida, no resuelta en la versión actual del sistema. |

### Usabilidad y accesibilidad

| ID | Requisito |
|----|-----------|
| RNF-04 | Un paciente sin conocimientos técnicos debe poder registrarse en autorecepción en **menos de 30 segundos y sin ayuda del personal**, desde un dispositivo táctil o un navegador estándar. El formulario tiene como máximo 2 campos. |
| RNF-09 | El código del turno en la pantalla de sala de espera debe mostrarse con un tamaño mínimo de **64 px**, legible desde los asientos de la sala, y el estado "llamado" debe indicarse con texto además de color. |

### Operación

| ID | Requisito |
|----|-----------|
| RNF-07 | El reinicio mensual del contador depende de una acción manual del equipo de desarrollo. Mientras tanto, los códigos crecen a lo largo del mes: con más de 70 pacientes por día, la categoría Consultas supera los 4 dígitos (ej. `C-2400`), lo que los hace menos legibles en la pantalla de sala de espera. Si el reinicio se omite, los códigos siguen creciendo; si se hace con la jornada en curso, puede repetir códigos el mismo día. Mejora propuesta: reinicio automático diario al ejecutar "Cerrar jornada". |
