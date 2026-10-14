# Stakeholders

> Relevado mediante 3 entrevistas presenciales realizadas en Fertya entre noviembre de 2025 y febrero de 2026 (ver documentación de la Práctica Profesionalizante II del grupo, carpeta 02).

---

## Paciente

**Tipo:** Externo
**Por qué es clave:** Es el usuario final del circuito de ingreso (autorecepción y seguimiento). Antes del sistema, era quien sufría directamente el circuito duplicado (doble fila entre Planta Baja y 4º piso), llegando en algunos casos tarde a su propia consulta pese a tener turno reservado. Al tratarse de un centro de medicina reproductiva, la espera y la incertidumbre agregan una carga emocional adicional a un proceso ya demandante, lo que hace más sensible este punto de contacto. Sus necesidades originan RF-01 a RF-04 (registrarse una sola vez), RF-13 a RF-15 (seguir el turno desde el celular) y RNF-04 (usarlo sin ayuda).
**Impacto: Alto.** Si el paciente no puede o no quiere autogestionarse, el sistema no reemplaza la fila física y el problema relevado sigue igual.

---

## Gerencia — Laura Gallo

**Tipo:** Interno
**Por qué es clave:** Es el punto de contacto institucional que habilitó la práctica y quien recibe los reclamos de los pacientes por tiempos de espera y demoras en el circuito de recepción. Es la responsable con mayor injerencia para autorizar un cambio de circuito y evaluar los beneficios de la propuesta a nivel institucional. Es la principal interesada en el reporte del día (RF-10, HU-06), que permite medir por primera vez los tiempos de espera.
**Impacto: Alto.** Sin su aprobación no se cambia el circuito de ingreso, y es quien decide si el sistema pasa a producción total.

---

## Coordinadora de Recepción — 4º piso (Berenice Panunzio)

**Tipo:** Interno
**Por qué es clave:** Supervisa el punto donde hoy se concentra la demora relevada: la segunda fila real de recepción en el 4º piso, donde se ubican los consultorios y la sala de espera principal. Su equipo es usuario directo del panel de recepción (`recepcion.html`) para llamar, devolver y finalizar turnos (RF-05 a RF-09).
**Impacto: Alto.** Su equipo opera el sistema todo el día; si el panel no le resulta ágil, vuelven al llamado verbal.

---

## Coordinadora de Recepción — Planta Baja (Gisela Rattia)

**Tipo:** Interno
**Por qué es clave:** Supervisa el primer punto de contacto del paciente al ingresar al edificio, donde hoy se produce la derivación informal (con o sin atención inicial) hacia el 4º piso. Su rol es clave para decidir si ese primer contacto en PB se mantiene con personal humano o se reemplaza por señalética que oriente al paciente a autogestionarse desde su celular. Su puesto y el del 4º piso pueden llamar el mismo turno a la vez, lo que origina el requisito RNF-06 y la limitación conocida LIM-01.
**Impacto: Alto.** Define cómo entra el paciente al circuito; de su decisión depende dónde se ubica la autorecepción.

---

## Profesionales de consultorio y laboratorio

**Tipo:** Interno
**Por qué es clave:** No usan el sistema directamente, pero reciben al paciente después de recepción. El relevamiento mostró pacientes que llegaban tarde a su propia consulta por la demora en la fila; un circuito más ágil impacta directamente en la puntualidad de sus agendas, sobre todo en laboratorio (7 a 11 hs, pacientes en ayunas).
**Impacto: Medio.** Se benefician del sistema pero no lo operan ni deciden sobre él.

---

## Personal de seguridad (Planta Baja)

**Tipo:** Interno
**Por qué es clave:** Actúa hoy como derivador informal del paciente entre PB y el 4º piso, sin protocolo formal relevado. Es relevante definir si conserva algún rol en el nuevo circuito o queda fuera de él.
**Impacto: Medio.** No opera el sistema, pero si sigue derivando pacientes sin indicarles la autorecepción, el circuito duplicado persiste.

---

## Área de sistemas / legal — Grupo Oroño

**Tipo:** Interno (a nivel de grupo, no de Fertya en particular)
**Por qué es clave:** El sistema almacena DNI y motivo de consulta en la nube (Firestore); se identificó como pendiente que esta área confirme el cumplimiento de la Ley 25.326 de Protección de Datos Personales antes de una puesta en producción total (RNF-03), y que asuma la continuidad/mantenimiento del sistema una vez finalizada la práctica profesional de los estudiantes.
**Impacto: Medio.** No interviene en la operación diaria, pero puede frenar la puesta en producción total si las limitaciones de seguridad LIM-03 y LIM-04 no se resuelven (RNF-03).

---

## Equipo de desarrollo (Grupo 1 — estudiantes)

**Tipo:** Interno (proveedor del sistema durante la práctica)
**Por qué es clave:** Desarrolló y hoy mantiene el sistema. Además es responsable de tareas operativas que el sistema no automatiza, como el reinicio mensual del contador de códigos (RF-03, LIM-02). Cuando termine la práctica profesional, esas tareas tienen que pasar al Área de sistemas de Grupo Oroño; si no, el sistema queda sin quien lo opere.
**Impacto: Alto mientras dure la práctica.** Su salida es un riesgo de continuidad.

---

## Firebase (Google) — Hosting + Firestore

**Tipo:** Sistema externo
**Por qué es clave:** Es la plataforma sobre la que corre todo el sistema: aloja las pantallas (Hosting), guarda los turnos y los contadores (Firestore) y notifica los cambios en tiempo real a todas las pantallas. Varias decisiones no son libres del equipo sino que dependen de cómo funciona Firebase: el tiempo real sin recargar (RNF-01), la seguridad basada en reglas de Firestore (RNF-03), la condición de carrera "gana la última escritura" (LIM-01) y el límite de 500 escrituras por lote que obliga a cerrar la jornada en lotes (CU-04). También es donde se guardan los datos personales, por lo que condiciona el análisis de la Ley 25.326.
**Impacto: Alto.** Si Firebase no está disponible, ninguna de las cuatro pantallas funciona (RNF-02).

---

## GitHub Actions

**Tipo:** Sistema externo
**Por qué es clave:** Publica automáticamente en Firebase Hosting cada cambio que se integra a la rama `main` (RNF-02). Un error subido a `main` llega directamente a las pantallas que usa Fertya, sin paso intermedio de revisión.
**Impacto: Bajo.** No interviene en la operación diaria; solo afecta cuándo y cómo llegan los cambios a producción.

---

## Tabla resumen

| Stakeholder | Tipo | Nivel de impacto | Por qué ese nivel |
|-------------|------|-------------------|-------------------|
| Paciente | Externo | Alto | Sin su adopción no se elimina la fila física. |
| Gerencia (Laura Gallo) | Interno | Alto | Autoriza el cambio de circuito y la puesta en producción. |
| Coordinadora Recepción 4º piso (Berenice Panunzio) | Interno | Alto | Su equipo opera el panel todo el día. |
| Coordinadora Recepción PB (Gisela Rattia) | Interno | Alto | Define cómo entra el paciente al circuito. |
| Profesionales de consultorio y laboratorio | Interno | Medio | Se benefician, pero no operan ni deciden. |
| Personal de seguridad | Interno | Medio | Puede sostener o romper el circuito nuevo con su derivación. |
| Área de sistemas / legal — Grupo Oroño | Interno | Medio | Puede frenar la producción total por datos personales. |
| Equipo de desarrollo (Grupo 1) | Interno | Alto (temporal) | Mantiene y opera tareas manuales; riesgo de continuidad. |
| Firebase (Google) | Sistema externo | Alto | Todo el sistema depende de su disponibilidad. |
| GitHub Actions | Sistema externo | Bajo | Solo afecta el despliegue de cambios. |
