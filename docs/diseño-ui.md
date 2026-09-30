# Diseño UI

_Presentar al menos un wireframe por pantalla o módulo relevante._
_Los wireframes en imagen o PDF van en `diagramas/wireframes/`; acá se documenta la justificación de cada uno._

> Los wireframes son de baja fidelidad (cajas y etiquetas), redibujados a partir de las pantallas reales del sistema desplegado en `https://gestor-fila.web.app`. Cada uno tiene su fuente PlantUML (`.puml`) y su imagen (`.png`). Todos los datos que aparecen (códigos, DNI, horarios) son de ejemplo, no de pacientes reales.

---
.
## Pantalla 1 — Autorecepción (`autorecepcion.html`)

**Wireframe:** `diagramas/wireframes/autorecepcion.png` (estado A: formulario; estado B: turno generado)

**Usuario:** paciente que llega a Fertya, muchas veces en ayunas y en la franja crítica de 7 a 11 hs (horario de laboratorio), sin conocimiento previo del sistema.

**Patrones de diseño utilizados:** Formulario corto de una sola pantalla + mensaje de confirmación en la misma pantalla (código grande + QR).

**Justificación:** RNF-04 exige que el paciente pueda usar la pantalla sin instrucciones. Por eso el formulario pide solo los dos datos imprescindibles (RF-01) y se resuelve en una sola pantalla. Con dos campos no hace falta un step-by-step, que solo agregaría pasos. El motivo se elige de una **lista desplegable** en vez de escribirse como texto libre: así el paciente no puede ingresar un motivo inexistente y el sistema siempre puede asignarle la categoría correcta al código (RF-03). La confirmación aparece en la misma pantalla y no en otra página. El paciente ve en el mismo lugar su código visible, que es lo que después busca en la sala de espera, y el QR para el seguimiento desde el celular (RF-04, HU-01). Por privacidad, la pantalla aclara que no se muestran nombres, solo el código (RNF-03).

**Formulario:**
- Cantidad de campos: 2 (DNI, motivo de la visita).
- Flujo: todo en una sola pantalla; al registrar, el formulario se limpia para el próximo paciente.
- Validaciones relevantes (RF-01b):
  - DNI numérico de 7 u 8 dígitos. Si no cumple, se muestra "Ingresá un DNI válido (7–8 dígitos)" y no se crea el turno.
  - Motivo obligatorio. Si falta, se muestra "Seleccioná un motivo".
  - Si falla la conexión con la base, se muestra "Error registrando. Avisá en recepción".
- El campo DNI abre el **teclado numérico** en dispositivos táctiles, lo que reduce errores de tipeo.

**Observación de mejora:** en la versión actual, el recuadro de confirmación se ve vacío antes de registrar el turno. Debería mostrarse solo después de un registro exitoso, para no confundir al paciente.

---

## Pantalla 2 — Panel de recepción (`recepcion.html`)

**Wireframe:** `diagramas/wireframes/recepcion.png` (incluye la ventana de confirmación de "Finalizar")

**Usuario:** recepcionistas de Planta Baja y del 4º piso (Recepción 1 a 4), trabajando en paralelo, con varios pacientes esperando.

**Patrones de diseño utilizados:** Tabla con acciones en línea + barra de herramientas superior + ventana modal de confirmación.

**Justificación:** El recepcionista necesita comparar varios turnos a la vez: quién llegó primero (hora de ingreso), en qué estado está cada uno y quién lo llamó (RF-05). Una **tabla** permite recorrer las columnas de un vistazo. Con más de 70 pacientes por día, una grilla de tarjetas obligaría a scrollear mucho más para ver la misma información. Las acciones van **en la misma fila** del turno para minimizar los clics por atención (RF-06, RF-07, RF-08, HU-02a/b/c). Además, **cambian según el estado**: un turno "en-fila" solo muestra "Llamar", y uno "llamado" muestra "Devolver" y "Finalizar". Así se evita que el recepcionista intente una transición que no corresponde. Las acciones sobre toda la jornada (exportar CSV, RF-10; cerrar jornada o limpiar el panel, RF-09 y RF-09b) van en la barra superior, separadas de las acciones por turno. El selector "Recepción 1–4" identifica quién llama (`llamadoPor`, RF-06) y queda guardado en el navegador, así el operador no tiene que elegirlo cada vez.

La **ventana modal de confirmación** se usa solo en las acciones que no se pueden deshacer desde el panel: "Finalizar" pide confirmación explícita. "Cerrar jornada" pregunta si finalizar los turnos en la base o solo limpiar la vista local, porque son dos acciones distintas (RF-09 vs. RF-09b). "Llamar" y "Devolver" no piden confirmación porque se revierten entre sí.

Cuando la pestaña del panel no está visible y llega un paciente nuevo, el sistema avisa con una notificación del navegador y hace titilar el título de la pestaña. Así el recepcionista no depende de estar mirando la tabla.

**Formulario:** No aplica (no hay carga de datos, solo acciones sobre turnos existentes).

**Observación de mejora:** la tabla muestra todos los turnos del día, incluidos los finalizados. Un filtro "solo activos" reduciría el ruido visual a medida que avanza la jornada.

---

## Pantalla 3 — Sala de espera (`pantalla.html`)

**Wireframe:** `diagramas/wireframes/pantalla.png`

**Usuario:** pacientes sentados en la sala de espera del 4º piso, mirando un monitor a varios metros de distancia, de rangos etarios amplios.

**Patrones de diseño utilizados:** Tarjetas (cards) en lista vertical + resaltado del estado "llamado" con borde de color y etiqueta de texto.

**Justificación:** Esta pantalla no tiene interacción: solo se mira de lejos. Cada tarjeta muestra un solo turno, con el código como dato principal (RF-11). Una tabla con varias columnas sería ilegible a distancia. El turno llamado se resalta con **borde de color y una etiqueta "LLAMANDO"**, y además indica desde qué recepción lo llaman (RF-12, HU-03). Así el paciente sabe a qué mostrador ir. Los datos se actualizan en tiempo real, sin recargar la página (RNF-01). No se muestran nombres ni DNI, solo el código (RNF-03).

**Formulario:** No aplica.

**Observación de mejora:** hoy la tarjeta muestra el motivo como código interno (`CON`, `LAB`) en lugar del nombre ("Consultas", "Laboratorio"), que el paciente sí reconoce.

---

## Pantalla 4 — Seguimiento (`seguimiento.html`)

**Wireframe:** `diagramas/wireframes/seguimiento.png` (estados: en fila, llamado, error)

**Usuario:** el paciente, desde su propio celular, después de escanear el QR. Puede estar en otro piso o afuera del edificio.

**Patrones de diseño utilizados:** Página de estado único (status page) + banner destacado al ser llamado + aviso sonoro opcional.

**Justificación:** El paciente consulta esta pantalla varias veces y por pocos segundos. Por eso muestra un único dato: el estado de su turno (RF-13, HU-04), sin menús ni navegación. Cuando lo llaman, aparece un **banner grande** con su código y la recepción que lo llama. El **sonido es opcional**: se activa con una casilla, porque el paciente puede estar en la sala de espera y no todos quieren que el celular suene (RF-15). Si el enlace no trae token o el token no existe, la pantalla muestra un mensaje de error claro sin exponer datos de ningún turno.

**Formulario:** No aplica (solo la casilla "Reproducir sonido al ser llamado").

---

## Consideraciones de accesibilidad

1. **Legibilidad a distancia en la sala de espera.** `pantalla.html` se mira desde varios metros por pacientes de todas las edades, incluidas personas con baja visión. Hoy el código del turno se muestra a 1.1rem (unos 18 px), un tamaño pensado para leer de cerca. Se define como criterio (RNF-09) que el código de la tarjeta se muestre a **un mínimo de 64 px** en el monitor de la sala, con texto oscuro sobre fondo claro. Este valor hay que validarlo en el lugar, a la distancia real de los asientos.
2. **No depender solo del color.** Cada recepción tiene su color (Recepción 1 a 4), pero el turno llamado también se identifica con la etiqueta de texto "LLAMANDO" y con la leyenda "Llamado por: Recepción N". Una persona con daltonismo puede identificar su turno y el mostrador sin distinguir los colores.
3. **Ayuda visible en el formulario de autorecepción.** La indicación "Solo números, sin puntos" queda visible debajo del campo DNI y no desaparece al escribir, a diferencia de un placeholder. Además, el campo abre el teclado numérico en pantallas táctiles.
4. **Aviso visual y sonoro en el seguimiento.** El llamado se comunica con un banner visual y, opcionalmente, con sonido: una persona con dificultad auditiva recibe el aviso por el banner, y una persona que no está mirando el celular, por el sonido.
