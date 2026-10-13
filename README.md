# Gestor de Filas Fertya — Grupo 1

> Materia: Diseño de Sistemas Web — Analista Funcional de Sistemas  
> Institución: Terciario Urquiza — Rosario  
> Docente: Pedernera Pablo  
> Cuatrimestre: 2.° 2026

## Integrantes

Ver [integrantes.md](integrantes.md)

## Descripción del proyecto

Gestor de Filas Fertya es un sistema web de gestión de turnos y filas de espera para pacientes, desarrollado sobre Firebase (Hosting + Firestore) y desplegado en `https://gestor-fila.web.app`.

El sistema reemplaza la gestión manual de la fila de pacientes por un circuito digital de cuatro pantallas:

* **Autorecepción** (`autorecepcion.html`): el paciente se autogestiona ingresando su DNI y motivo de visita; el sistema genera un código de turno visible (por ej. `C-001`, `L-002`) y un QR/token de seguimiento.
* **Recepción** (`recepcion.html`): panel operativo para el personal de recepción, que llama, devuelve o finaliza turnos en tiempo real y puede exportar el reporte del día en CSV.
* **Pantalla de sala de espera** (`pantalla.html`): muestra en tiempo real los turnos en fila y los turnos llamados, para visualización pública en un monitor.
* **Seguimiento** (`seguimiento.html`): permite a cada paciente seguir el estado de su propio turno desde su celular mediante un token único en la URL.

Los datos de los turnos se almacenan en Firestore (colección `turnos\_fertya`), con contadores atómicos por categoría de motivo (colección `counters`) para generar los códigos secuenciales. El despliegue a producción es automático: cada push a la rama `main` dispara un workflow de GitHub Actions que publica el contenido de `public/` en Firebase Hosting.

## Caso de estudio

**Fertya — Medicina Reproductiva** (Grupo Oroño), Rosario. Centro médico privado especializado en diagnóstico y tratamiento de infertilidad, con sede principal en Rioja 2282 (edificio de 6 plantas) y una sede secundaria en San Nicolás. Recibe más de 70 pacientes por día entre tratamientos y controles, con la franja más crítica entre las 7 y las 11 hs (horario de laboratorio, pacientes en ayunas).

**Problema relevado** (evidencia: 3 entrevistas presenciales — Laura Gallo, Gerente; Berenice Panunzio, Coordinadora de Recepción del 4º piso; y Gisela Rattia, Coordinadora de Recepción de Planta Baja): antes del sistema, el ingreso del paciente era un **circuito duplicado**. El paciente tenía un primer contacto informal en Planta Baja (mostrador o personal de seguridad) que no resolvía su recepción, y recién se recepcionaba de verdad al llegar al 4º piso — donde volvía a hacer fila, aunque ya tuviera un turno reservado. Esto generaba pacientes que llegaban tarde a su propia consulta por la demora en la fila de recepción, incomodidad por el espacio reducido del pasillo del 4º piso, y ningún registro objetivo de quién había llegado primero ni de cuánto tiempo esperaba realmente cada paciente. El Gestor de Filas Fertya resuelve esto unificando el punto de registro: el paciente queda registrado una sola vez (`autorecepcion.html`), sin importar en qué piso se lo atienda después.

## Entregas

|Entrega|Descripción|Fecha|Estado|
|-|-|-|-|
|EP-01|Presentación preliminar (Stakeholders + Requisitos)|19/08/2026|Entregada|
|EP-02 (Entrega intermedia)|Requisitos + Historias de Usuario + Casos de Uso completos, con revisión en vivo|16/09/2026|Entregada|
|Final|Presentación final grupal (Parcial, Parte A)|13/10/2026|En curso — repo completo, en revisión final|

### Cronograma detallado de trabajo

|Fecha|Actividad|
|-|-|
|Mar 08/09|Cierre de Requisitos y HU + arranque del diagrama general de Casos de Uso|
|Mié 09/09|Trabajo en grupo: Casos de Uso — diagrama general completo + arranque de CUs desarrollados|
|Mar 15/09|Trabajo en grupo: cierre de Casos de Uso (precondiciones, postcondiciones, secuencia, excepciones)|
|Mié 16/09|**ENTREGA INTERMEDIA** — Requisitos + HU + Casos de Uso completos, con revisión en vivo|
|Mar 22/09|Revisión en vivo de wireframes (Diseño UI)|
|Mié 23/09|Trabajo en grupo: Modelo ER|
|Mar 29/09|Trabajo en grupo: wireframes (Diseño UI)|
|Mié 30/09|Consulta general y repaso|
|Mar 06/10|Consulta final del repo + repaso para el cuestionario individual|
|Mié 07/10|Últimos ajustes: README, integrantes.md, estructura del repo completa|
|Mar 13/10|**PRESENTACIÓN FINAL GRUPAL** (Parcial, Parte A)|
|Mié 14/10|Cierre del cuatrimestre + repaso para el Cuestionario individual|

## Devoluciones atendidas

Cambios hechos a partir de la devolución del 30/09/2026 ([`feedback/2026-09-30.md`](feedback/2026-09-30.md)):

| Observación del docente | Qué cambiamos | Dónde |
|---|---|---|
| RNF-06 y RNF-07 son limitaciones, no requisitos. | Se reescribieron como requisitos ("el sistema debe…") y las limitaciones actuales pasaron a una sección aparte (LIM-01 a LIM-05). | `docs/requisitos.md` |
| HU-04 (criterio 3) contradice a RF-14 sobre el token consumido. | Se alineó con RF-14: el token consumido no bloquea el seguimiento. | `docs/historias-de-usuario.md` |
| HU-01 (criterio 2) y CU-01 contemplan un "motivo desconocido" que la interfaz no permite. | Se reemplazó por la validación de DNI y motivo, y por la falla al guardar ("Error registrando. Avisá en recepción"). | `docs/historias-de-usuario.md`, `docs/casos-de-uso.md` |
| Validación INVEST casi vacía. | Se completaron las observaciones de las 9 historias, con dependencias reales y verificación contra RF/RNF. | `docs/historias-de-usuario.md` |
| Faltan HU de cerrar jornada, exportar CSV y aviso por pestaña no visible; HU-02c sin confirmación. | Se agregaron HU-05, HU-06 y HU-07, y la confirmación en HU-02c. | `docs/historias-de-usuario.md` |
| Solo dos casos de uso desarrollados. | Se desarrollaron CU-03 (Consultar seguimiento) y CU-04 (Cerrar jornada). | `docs/casos-de-uso.md` |
| Excepciones pocas; E1 de CU-02 es un error del usuario. | Se agregaron las excepciones ya identificadas en Diseño UI y otras reales; los errores del usuario pasaron a flujos alternativos. Se mantuvo sin `include` la relación entre "Cerrar jornada" y "Finalizar turno", con la justificación escrita. | `docs/casos-de-uso.md`, `diagramas/casos-de-uso.puml` |
| Rendimiento de CU-01 "en pocos segundos". | Se usa el mismo valor de RNF-08: menos de 3 s. | `docs/casos-de-uso.md` |
| ER sin hora de finalización; redundancias sin explicar; falta justificar la ausencia de Paciente. | Se agregó `horaFinalizado` y las decisiones 3 a 7 (Paciente, Recepcionista, hora de ingreso, último llamado y motivo/categoría). | `docs/er-modelo.md`, `diagramas/er.puml` |

Cambios hechos a partir de la devolución del 06/10/2026 ([`feedback/2026-10-06.md`](feedback/2026-10-06.md)):

| Observación del docente | Qué cambiamos | Dónde |
|---|---|---|
| El criterio 3 de HU-02a describe el error actual, no lo que debe pasar. | Ahora dice lo que exige RNF-06 (se registra solo el primer llamado y se avisa al segundo operador); la limitación actual queda en LIM-01. | `docs/historias-de-usuario.md` |
| Firebase como actor no está aplicado parejo en el diagrama. | Se conectó con todos los casos de uso que leen o escriben turnos, y se justificó por qué se modela como actor. | `diagramas/casos-de-uso.puml`, `docs/casos-de-uso.md` |
| El `extend` de "Devolver turno" sobre "Llamar turno" es discutible. | "Devolver turno" pasó a ser un caso de uso independiente del recepcionista, con la justificación escrita. | `diagramas/casos-de-uso.puml`, `docs/casos-de-uso.md` |
| El ER no tiene `horaFinalizado`; redundancias sin explicar; falta la decisión sobre Paciente. | Ver la fila del ER en la tabla del 30/09. | `docs/er-modelo.md`, `diagramas/er.puml` |
| DoR: ítem 6 no verificable; faltan ítems de dependencias y RNF; no dice qué cambió. | Checklist de 8 ítems con dependencias (ítem 5) y RNF (ítem 6), y una sección con los cambios que salieron del análisis. | `DoR.md` |
| Slicing: la Historia 4 no es un slice propio; historias con un solo criterio. | La Historia 4 pasó a "Compartir el comprobante"; todas tienen al menos dos criterios (la 3 suma el caso de saldo exacto). | `slicing.md` |

## Estructura del repositorio

```
/
├── README.md
├── integrantes.md
├── DoR.md
├── slicing.md
├── RECURSOS.md
├── feedback/
├── docs/
│   ├── requisitos.md
│   ├── historias-de-usuario.md
│   ├── casos-de-uso.md
│   ├── er-modelo.md
│   ├── diseño-ui.md
│   └── stakeholders.md
├── diagramas/
│   ├── casos-de-uso.puml
│   ├── er.puml
│   └── wireframes/
│       ├── autorecepcion.puml / .png
│       ├── recepcion.puml / .png
│       ├── pantalla.puml / .png
│       └── seguimiento.puml / .png
└── cuestionario/
```

## Instrucciones operativas

* Un integrante del grupo es responsable de subir los cambios al repositorio.
* Completar `integrantes.md` antes de la primera entrega.
* Mantener los archivos en la carpeta correspondiente según la estructura indicada.
* Los diagramas deben entregarse en formato PlantUML (`.puml`). Se pueden visualizar en [plantuml.com](https://www.plantuml.com/plantuml/uml/).

