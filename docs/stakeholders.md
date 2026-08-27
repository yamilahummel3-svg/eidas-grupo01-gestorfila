# Stakeholders

_Identificar y justificar las partes interesadas relevantes para el sistema._
_Para cada una: describir su rol y por qué es clave para el proyecto._

> Relevado mediante 3 entrevistas presenciales realizadas en Fertya entre noviembre de 2025 y febrero de 2026 (ver documentación de la Práctica Profesionalizante II del grupo, carpeta 02).

---

## Paciente

**Tipo:** Externo
**Por qué es clave:** Es el usuario final del circuito de ingreso (autorecepción y seguimiento). Antes del sistema, era quien sufría directamente el circuito duplicado (doble fila entre Planta Baja y 4º piso), llegando en algunos casos tarde a su propia consulta pese a tener turno reservado. Al tratarse de un centro de medicina reproductiva, la espera y la incertidumbre agregan una carga emocional adicional a un proceso ya demandante, lo que hace más sensible este punto de contacto.

---

## Gerencia — Laura Gallo

**Tipo:** Interno
**Por qué es clave:** Es el punto de contacto institucional que habilitó la práctica y quien recibe los reclamos de los pacientes por tiempos de espera y demoras en el circuito de recepción. Es la responsable con mayor injerencia para autorizar un cambio de circuito y evaluar los beneficios de la propuesta a nivel institucional.

---

## Coordinadora de Recepción — 4º piso (Berenice Panunzio)

**Tipo:** Interno
**Por qué es clave:** Supervisa el punto donde hoy se concentra la demora relevada: la segunda fila real de recepción en el 4º piso, donde se ubican los consultorios y la sala de espera principal. Es usuaria directa del panel de recepción (`recepcion.html`) para llamar, devolver y finalizar turnos.

---

## Coordinadora de Recepción — Planta Baja (Gisela Rattia)

**Tipo:** Interno
**Por qué es clave:** Supervisa el primer punto de contacto del paciente al ingresar al edificio, donde hoy se produce la derivación informal (con o sin atención inicial) hacia el 4º piso. Su rol es clave para decidir si ese primer contacto en PB se mantiene con personal humano o se reemplaza por señalética que oriente al paciente a autogestionarse desde su celular.

---

## Personal de seguridad (Planta Baja)

**Tipo:** Interno
**Por qué es clave:** Actúa hoy como derivador informal del paciente entre PB y el 4º piso, sin protocolo formal relevado. Es relevante definir si conserva algún rol en el nuevo circuito o queda fuera de él.

---

## Área de sistemas / legal — Grupo Oroño

**Tipo:** Interno (a nivel de grupo, no de Fertya en particular)
**Por qué es clave:** El sistema almacena DNI y motivo de consulta en la nube (Firestore); se identificó como pendiente que esta área confirme el cumplimiento de la Ley 25.326 de Protección de Datos Personales antes de una puesta en producción total, y que asuma la continuidad/mantenimiento del sistema una vez finalizada la práctica profesional de los estudiantes.

---

## Tabla resumen

| Stakeholder | Tipo | Nivel de impacto |
|-------------|------|-------------------|
| Paciente | Externo | Alto |
| Gerencia (Laura Gallo) | Interno | Alto |
| Coordinadora Recepción 4º piso (Berenice Panunzio) | Interno | Alto |
| Coordinadora Recepción PB (Gisela Rattia) | Interno | Alto |
| Personal de seguridad | Interno | Medio |
| Área de sistemas / legal — Grupo Oroño | Interno | Medio |
