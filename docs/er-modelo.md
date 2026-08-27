# Modelo Entidad-Relación

## Diagrama

Incluido en `diagramas/er.puml`. Visualizar en [plantuml.com](https://www.plantuml.com/plantuml/uml/).

> **Nota:** el sistema real usa Firestore (base NoSQL orientada a documentos), no una base relacional. El modelo siguiente traduce la estructura de las colecciones `turnos_fertya` y `counters` a un esquema entidad-relación con fines de análisis y documentación, tal como pide la consigna.

## Entidades

| Entidad | Descripción | Relaciones clave |
|---------|-------------|-------------------|
| Turno | Representa el turno de un paciente en la fila, desde que se autogestiona hasta que finaliza. | Pertenece a una Categoría; tiene 0..N Llamadas. |
| Categoria | Categoría del motivo de visita (letra), con su contador secuencial para generar el código visible. | Agrupa 0..N Turnos. |
| Llamada | Registro histórico de cada vez que un turno fue llamado (soporta el caso de "llamar → devolver → volver a llamar"). | Pertenece a un Turno. |

## Descripción de atributos principales

### Turno

- `id_turno` (PK): identificador del documento en Firestore.
- `dni`: DNI del paciente que generó el turno.
- `motivoCodigo`: motivo de la visita, tal como lo ingresó el paciente.
- `codigoVisible`: código secuencial mostrado al paciente (ej. `C-001`).
- `token`: identificador único (UUID) usado para el seguimiento remoto del turno.
- `estado`: estado actual (`en-fila`, `llamado`, `finalizado`).
- `createdAt` / `horaIngreso`: momento de creación del turno.
- `horaLlamado`: momento del último llamado.
- `llamadoPor`: identificador de quién realizó el último llamado.
- `tokenConsumed` / `tokenCreatedAt` / `tokenConsumedAt` / `tokenConsumedBy`: control de uso del token de seguimiento.
- `id_categoria` (FK): categoría a la que pertenece el turno.

### Categoria

- `id_categoria` (PK): letra de la categoría (`C`, `L`, `E`, `T`, `P`, `O`).
- `seq`: contador secuencial usado para generar el próximo `codigoVisible` de esa categoría.

### Llamada

- `id_llamada` (PK): identificador del evento de llamado.
- `id_turno` (FK): turno al que pertenece.
- `by`: quién realizó el llamado.
- `at`: momento del llamado.

## Decisiones de diseño

### Decisión 1 — Separar "Llamada" como entidad propia

En la implementación real, el historial de llamadas se guarda embebido como un arreglo dentro del propio documento de Turno (`llamadas: [{by, at}, ...]`), lo cual es eficiente en Firestore porque evita una consulta adicional. Para este modelo entidad-relación se decidió separarlo como entidad propia, porque conceptualmente representa un historial de eventos con cardinalidad variable (un turno puede ser llamado, devuelto y llamado de nuevo más de una vez), y modelarlo así deja más claro el ciclo de vida del turno de cara al análisis. Se evaluó dejarlo como un atributo simple (`ultimoLlamado`), pero se descartó porque se perdería el historial completo, que es necesario para trazabilidad.

### Decisión 2 — Contador atómico por categoría en vez de calcular el código al vuelo

Se decidió mantener una entidad Categoria con un contador (`seq`) independiente del turno, en lugar de calcular el código visible contando los turnos existentes al momento de mostrarlo. La alternativa descartada (contar turnos de la categoría en el momento) no garantiza atomicidad: si dos pacientes se autogestionan casi al mismo tiempo, podrían recibir el mismo código. Con un contador atómico por categoría se evita esa condición de carrera.
