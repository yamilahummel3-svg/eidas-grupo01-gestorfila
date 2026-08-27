# Diseño UI

_Presentar al menos un wireframe por pantalla o módulo relevante._
_Los wireframes en imagen o PDF van en `diagramas/wireframes/`; acá se documenta la justificación de cada uno._

---

## Pantalla 1 — Autorecepción (`autorecepcion.html`)

**Wireframe:** `diagramas/wireframes/autorecepcion.png` _(pendiente: agregar captura de la pantalla real o boceto)_

**Patrones de diseño utilizados:** Formulario corto de una sola pantalla + pantalla de confirmación con código grande y QR.

**Justificación:** Los pacientes que llegan a una clínica no necesariamente están familiarizados con apps; un formulario de solo dos campos (DNI y motivo) reduce la fricción y la posibilidad de error al autogestionarse. Mostrar el código y el QR en tamaño grande permite que el paciente lo identifique con claridad sin ayuda del personal.

**Formulario:**
- Cantidad de campos: 2 (DNI, motivo de visita).
- Flujo: todo en una pantalla.
- Validaciones relevantes: DNI numérico y obligatorio; motivo obligatorio.

---

## Pantalla 2 — Recepción (`recepcion.html`)

**Wireframe:** `diagramas/wireframes/recepcion.png` _(pendiente)_

**Patrones de diseño utilizados:** Tabla con acciones en línea (Llamar / Devolver / Finalizar) + panel de control (cerrar jornada, exportar CSV).

**Justificación:** El personal de recepción necesita ver de un vistazo todos los turnos activos y actuar rápido, sin navegar entre pantallas. Una tabla con acciones directas por fila minimiza la cantidad de clics por atención, algo crítico cuando hay varios pacientes esperando.

**Formulario:** No aplica (no hay carga de datos, solo acciones sobre turnos existentes).

---

## Pantalla 3 — Sala de espera (`pantalla.html`)

**Wireframe:** `diagramas/wireframes/pantalla.png` _(pendiente)_

**Patrones de diseño utilizados:** Tarjetas (cards) grandes, resaltado de estado.

**Justificación:** Esta pantalla se visualiza a distancia en un monitor de sala de espera. Las tarjetas grandes y el resaltado del turno "llamado" permiten que los pacientes identifiquen su turno sin necesidad de acercarse o preguntar al personal.

**Formulario:** No aplica.

---

## Pantalla 4 — Seguimiento (`seguimiento.html`)

**Wireframe:** `diagramas/wireframes/seguimiento.png` _(pendiente)_

**Patrones de diseño utilizados:** Pantalla de estado único (status page).

**Justificación:** Pensada para uso en el celular del paciente; muestra un solo dato relevante (el estado de su turno), sin distraer con otros elementos, ya que se espera que el paciente la consulte brevemente y de forma repetida.

**Formulario:** No aplica.

---

## Consideraciones de accesibilidad

- En `pantalla.html`, al visualizarse en un monitor a distancia y dirigirse a una población de pacientes con rangos etarios amplios, el tamaño de fuente y el contraste de las tarjetas de turno deben ser suficientes para personas con baja visión — esto se marca como punto a verificar/ajustar en la implementación actual, no como algo ya validado.