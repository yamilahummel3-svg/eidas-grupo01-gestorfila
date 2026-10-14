# Modelo Entidad-Relación

## Diagrama

Incluido en `diagramas/er.puml`. Visualizar en [plantuml.com](https://www.plantuml.com/plantuml/uml/).

> **Nota:** el sistema real usa Firestore (base NoSQL orientada a documentos), no una base relacional. El modelo siguiente traduce la estructura de las colecciones `turnos_fertya` y `counters` a un esquema entidad-relación con fines de análisis y documentación, tal como pide la consigna.

## Entidades

| Entidad | Descripción | Relaciones clave |
|---------|-------------|-------------------|
| Turno | Representa el turno de un paciente en la fila, desde que se autogestiona hasta que finaliza (CU-01 a CU-04). | Pertenece a una Categoría (N:1); tiene 0..N Llamadas (1:N). |
| Categoria | Categoría del motivo de visita (letra), con su contador secuencial para generar el código visible (RF-03). | Agrupa 0..N Turnos. |
| Llamada | Registro histórico de cada vez que un turno fue llamado (soporta el caso de "llamar → devolver → volver a llamar", CU-02 E1 y E2). | Pertenece a un Turno. |

## Descripción de atributos principales

### Turno

- `id_turno` (PK): identificador del documento en Firestore. Es el identificador inequívoco del turno (ver Decisión 2).
- `dni`: DNI del paciente que generó el turno (RF-01).
- `motivoCodigo`: código del motivo elegido de la lista (`CON`, `LAB`, `ES`, `CT`, `OT`).
- `codigoVisible`: código secuencial mostrado al paciente (ej. `C-001`, RF-03).
- `token`: identificador único (UUID) usado para el seguimiento remoto del turno (RF-02, RNF-05).
- `estado`: estado actual, con tres valores posibles: `en-fila`, `llamado`, `finalizado`.
- `horaIngreso`: momento de creación del turno.
- `horaLlamado`: momento del último llamado (RF-06). Se vacía si el turno se devuelve (RF-07).
- `llamadoPor`: puesto de recepción que hizo el último llamado (Recepción 1 a 4).
- `horaFinalizado`: momento de finalización, individual (RF-08) o por cierre de jornada (RF-09).
- `tokenConsumed` / `tokenCreatedAt` / `tokenConsumedAt` / `tokenConsumedBy`: control de uso del token de seguimiento (RF-14).
- `id_categoria` (FK): categoría a la que pertenece el turno.

Con `horaIngreso`, `horaLlamado` y `horaFinalizado` se pueden calcular las dos métricas que no existían antes del sistema: **tiempo de espera** (llamado − ingreso) y **duración de la atención** (finalizado − llamado). Son las columnas que exporta el reporte CSV (RF-10, HU-06).

### Categoria

- `id_categoria` (PK): letra de la categoría. En uso: `C` (Consultas), `L` (Laboratorio), `E` (Espermograma), `T` (Análisis / Ecografías) y `O` (Otras gestiones).
- `seq`: contador secuencial usado para generar el próximo `codigoVisible` de esa categoría.

### Llamada

- `id_llamada` (PK): identificador del evento de llamado.
- `id_turno` (FK): turno al que pertenece.
- `by`: puesto de recepción que realizó el llamado.
- `at`: momento del llamado.

## Decisiones de diseño

### Decisión 1 — Separar "Llamada" como entidad propia

En la implementación real, el historial de llamadas se guarda embebido como un arreglo dentro del propio documento de Turno (`llamadas: [{by, at}, ...]`), lo cual es eficiente en Firestore porque evita una consulta adicional. Para este modelo entidad-relación se decidió separarlo como entidad propia, porque conceptualmente representa un historial de eventos con cardinalidad variable (un turno puede ser llamado, devuelto y llamado de nuevo más de una vez), y modelarlo así deja más claro el ciclo de vida del turno de cara al análisis. Se evaluó dejarlo como un atributo simple (`ultimoLlamado`), pero se descartó porque se perdería el historial completo, que es necesario para trazabilidad.

> **Hallazgo no confirmado:** el código arma el historial con una marca de hora del servidor dentro de un arreglo, algo que Firestore no admite. Si la escritura falla, la app guarda el llamado sin historial (CU-02, E2). No pudo confirmarse en la consola de Firestore antes de la entrega; si se confirma, esta entidad refleja el diseño buscado pero no el dato que hoy se guarda.

### Decisión 2 — Contador atómico por categoría en vez de calcular el código al vuelo

Se decidió mantener una entidad Categoria con un contador (`seq`) independiente del turno, en lugar de calcular el código visible contando los turnos existentes al momento de mostrarlo. La alternativa descartada (contar turnos de la categoría en el momento) no garantiza atomicidad: si dos pacientes se autogestionan casi al mismo tiempo, podrían recibir el mismo código. Con un contador atómico por categoría se evita esa condición de carrera.

El contador `seq` se reinicia manualmente una vez por mes (RF-03, LIM-02). Se evaluó reiniciarlo en cada "Cerrar jornada", pero en la versión actual se mantiene manual. La consecuencia es que `codigoVisible` no es único en el tiempo: el mismo código se repite de un mes a otro. Por eso el identificador inequívoco del turno es `id_turno` y no el código visible.

### Decisión 3 — El DNI como atributo del turno, sin entidad Paciente

Se evaluó crear una entidad Paciente (con DNI como PK) relacionada 1:N con Turno. Se descartó porque el sistema no gestiona pacientes: no guarda nombre, historia ni datos de contacto, solo necesita saber quién llegó para que recepción lo identifique. Una entidad Paciente invitaría a guardar más datos personales de los necesarios, justo lo que hay que minimizar por la Ley 25.326 (RNF-03). Como consecuencia, el sistema no detecta si un mismo DNI se registró dos veces en el día (CU-01, E5).

### Decisión 4 — Puesto de recepción como texto, sin entidad Recepcionista

`llamadoPor` guarda el puesto (Recepción 1 a 4) como texto, en lugar de una FK a una entidad Recepcionista. Se descartó la entidad porque el panel no tiene inicio de sesión (LIM-03): el sistema no sabe qué persona está operando, solo qué puesto eligió en el selector. Modelar una entidad Recepcionista daría a entender una trazabilidad por persona que el sistema real no tiene.

### Decisión 5 — Una sola marca de hora de ingreso

En la base real, cada turno guarda dos campos con el mismo valor al crearse: `createdAt` y `horaIngreso`. En el modelo se conserva solo `horaIngreso`, porque duplicar el dato no aporta información y abre la puerta a que los dos valores diverjan. En la implementación, `createdAt` se usa para filtrar y ordenar los turnos del día.

### Decisión 6 — Datos del último llamado repetidos en Turno (desnormalización)

`horaLlamado` y `llamadoPor` en Turno repiten los datos del último registro de Llamada. Es una redundancia deliberada: el panel de recepción, la pantalla de sala y el seguimiento necesitan saber en todo momento quién llamó y a qué hora, y leerlo del propio turno evita recorrer el historial en cada actualización en tiempo real (RNF-01, menos de 2 segundos). La alternativa descartada, calcular el último llamado a partir de Llamada, es la forma normalizada, pero obligaría a cada pantalla a ordenar el historial de cada turno en cada cambio. El costo de la decisión es que los dos datos pueden quedar desalineados: por ejemplo, al devolver un turno se borra `horaLlamado` pero el historial conserva el llamado, lo que es correcto (el turno ya no está llamado, pero sí fue llamado).

### Decisión 7 — Motivo y categoría como dos datos distintos

`motivoCodigo` guarda lo que el paciente eligió en la lista (`CON`, `LAB`, `ES`, `CT`, `OT`) e `id_categoria` guarda la letra del contador (`C`, `L`, `E`, `T`, `O`). Hoy cada motivo corresponde a una sola letra, por lo que parecen el mismo dato. Se mantienen separados porque cumplen funciones distintas: el motivo es la información que ve recepción en el panel y que sale en el reporte CSV (RF-05, RF-10), y la categoría solo define la numeración del código visible (RF-03). Separarlos permite, por ejemplo, agregar un motivo nuevo que comparta numeración con uno existente sin crear un contador nuevo. La alternativa descartada, derivar el motivo de la letra, perdería esa flexibilidad y ataría el reporte a la numeración.
