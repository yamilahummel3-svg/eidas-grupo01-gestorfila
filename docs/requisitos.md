# Requisitos del sistema

## Descripción del sistema

Gestor de Filas Fertya es un sistema web para la gestión digital de la fila de pacientes de Fertya (medicina reproductiva). Reemplaza el llamado manual/verbal de pacientes por un circuito con autogestión de turno, panel de recepción en tiempo real, pantalla pública de sala de espera y seguimiento remoto vía token. Está desplegado sobre Firebase (Hosting + Firestore), con autenticación anónima y despliegue automático desde GitHub Actions.

## Requisitos funcionales

### Módulo 1 — Autorecepción de pacientes

| ID | Requisito |
|----|-----------|
| RF-01 | El sistema debe permitir al paciente ingresar su DNI y motivo de visita para generar un turno. |
| RF-02 | El sistema debe generar un token único (UUID) asociado a cada turno creado. |
| RF-03 | El sistema debe asignar un código visible secuencial según la categoría del motivo (ej. `C-001`, `L-002`), incrementando un contador atómico por categoría. |
| RF-04 | El sistema debe mostrar al paciente el código visible generado junto a un código QR para su seguimiento. |

### Módulo 2 — Recepción (gestión de turnos)

| ID | Requisito |
|----|-----------|
| RF-05 | El sistema debe mostrar al personal de recepción los turnos activos en tiempo real, ordenados por fecha de creación. |
| RF-06 | El sistema debe permitir cambiar el estado de un turno a "llamado", registrando la hora y el usuario que realizó el llamado. |
| RF-07 | El sistema debe permitir devolver un turno de "llamado" a "en-fila". |
| RF-08 | El sistema debe permitir finalizar un turno individualmente. |
| RF-09 | El sistema debe permitir, al cerrar la jornada, finalizar de forma masiva todos los turnos que quedaron en estado "en-fila". |
| RF-10 | El sistema debe permitir exportar un reporte de los turnos del día en formato CSV. |

### Módulo 3 — Pantalla de sala de espera

| ID | Requisito |
|----|-----------|
| RF-11 | El sistema debe mostrar en tiempo real los turnos en estado "en-fila" y "llamado". |
| RF-12 | El sistema debe resaltar visualmente el/los turno(s) en estado "llamado". |

### Módulo 4 — Seguimiento del paciente

| ID | Requisito |
|----|-----------|
| RF-13 | El sistema debe permitir a un paciente consultar el estado de su propio turno accediendo mediante un token único incluido en la URL. |
| RF-14 | El sistema debe marcar el token como consumido tras su primer uso, registrando la fecha/hora de consumo. |
| RF-15 | El sistema debe notificar al paciente (sonido opcional) cuando su turno pasa a estado "llamado". |

## Requisitos no funcionales

### Rendimiento y disponibilidad

| ID | Requisito |
|----|-----------|
| RNF-01 | Las actualizaciones de estado de los turnos deben reflejarse en tiempo real en el panel de recepción y en la pantalla de sala de espera (listeners de Firestore, sin necesidad de recargar la página). |
| RNF-02 | El sistema debe estar disponible públicamente vía Firebase Hosting, con despliegue automático ante cada cambio integrado a la rama principal del repositorio. |

### Seguridad y usabilidad

| ID | Requisito |
|----|-----------|
| RNF-03 | El acceso a Firestore se realiza mediante autenticación anónima; el acceso de lectura/escritura a la colección de turnos (que incluye DNI) debe estar acotado mediante reglas de seguridad de Firestore. Al almacenarse datos personales de pacientes en la nube, se identificó como pendiente confirmar con el área de sistemas/legal de Grupo Oroño el cumplimiento de la Ley 25.326 de Protección de Datos Personales antes de una puesta en producción total. _(Pendiente de verificar/documentar las reglas de Firestore vigentes.)_ |
| RNF-04 | La pantalla de autorecepción debe poder ser utilizada por pacientes sin conocimientos técnicos, en un dispositivo táctil o navegador estándar, sin instrucciones adicionales. |
| RNF-05 | El token de seguimiento debe ser de un solo uso relevante (se marca como consumido) para reducir el acceso reiterado o no previsto a la información del turno. |
