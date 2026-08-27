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
- **Autorecepción** (`autorecepcion.html`): el paciente se autogestiona ingresando su DNI y motivo de visita; el sistema genera un código de turno visible (por ej. `C-001`, `L-002`) y un QR/token de seguimiento.
- **Recepción** (`recepcion.html`): panel operativo para el personal de recepción, que llama, devuelve o finaliza turnos en tiempo real y puede exportar el reporte del día en CSV.
- **Pantalla de sala de espera** (`pantalla.html`): muestra en tiempo real los turnos en fila y los turnos llamados, para visualización pública en un monitor.
- **Seguimiento** (`seguimiento.html`): permite a cada paciente seguir el estado de su propio turno desde su celular mediante un token único en la URL.

Los datos de los turnos se almacenan en Firestore (colección `turnos_fertya`), con contadores atómicos por categoría de motivo (colección `counters`) para generar los códigos secuenciales. El despliegue a producción es automático: cada push a la rama `main` dispara un workflow de GitHub Actions que publica el contenido de `public/` en Firebase Hosting.

## Caso de estudio
**Fertya — Medicina Reproductiva** (Grupo Oroño), Rosario. Centro médico privado especializado en diagnóstico y tratamiento de infertilidad, con sede principal en Rioja 2282 (edificio de 6 plantas) y una sede secundaria en San Nicolás. Recibe más de 70 pacientes por día entre tratamientos y controles, con la franja más crítica entre las 7 y las 11 hs (horario de laboratorio, pacientes en ayunas).

**Problema relevado** (evidencia: 3 entrevistas presenciales — Laura Gallo, Gerente; Berenice Panunzio, Coordinadora de Recepción del 4º piso; y Gisela Rattia, Coordinadora de Recepción de Planta Baja): antes del sistema, el ingreso del paciente era un **circuito duplicado**. El paciente tenía un primer contacto informal en Planta Baja (mostrador o personal de seguridad) que no resolvía su recepción, y recién se recepcionaba de verdad al llegar al 4º piso — donde volvía a hacer fila, aunque ya tuviera un turno reservado. Esto generaba pacientes que llegaban tarde a su propia consulta por la demora en la fila de recepción, incomodidad por el espacio reducido del pasillo del 4º piso, y ningún registro objetivo de quién había llegado primero ni de cuánto tiempo esperaba realmente cada paciente. El Gestor de Filas Fertya resuelve esto unificando el punto de registro: el paciente queda registrado una sola vez (`autorecepcion.html`), sin importar en qué piso se lo atienda después.

## Entregas
| Entrega | Descripción | Fecha | Estado |
|---------|-------------|-------|--------|
| EP-01 | Presentación preliminar (Stakeholders + Requisitos) | 19/08/2026 | [confirmar] |
| EP-02 | | [completar] | |
| Final | Versión definitiva | [completar] | |

## Estructura del repositorio
```
/
├── README.md
├── integrantes.md
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
└── cuestionario/
```

## Instrucciones operativas
- Un integrante del grupo es responsable de subir los cambios al repositorio.
- Completar `integrantes.md` antes de la primera entrega.
- Mantener los archivos en la carpeta correspondiente según la estructura indicada.
- Los diagramas deben entregarse en formato PlantUML (`.puml`). Se pueden visualizar en [plantuml.com](https://www.plantuml.com/plantuml/uml/).
