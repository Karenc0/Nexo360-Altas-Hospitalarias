# Automatizaciones: Flujo Final Caja → Admisión
**Proyecto:** NEXO 360° — Clínica San Felipe
**Etapas cubiertas:** 08. Egreso Físico (Ejecutivas) → 09. Cama Liberada (Admisión)
**Responsables:** Ariana Ibarra (Caja) · Valentino Esteban (Admisión)
**Creado:** 2026-03-16

---

## Contexto del flujo

```
[07. Alta Aprobada] → [08. Egreso Físico (Ejecutivas)] → [09. Cama Liberada (Admisión)]
                              ↑ Caja cobra                       ↑ Admisión desocupa
```

El paciente ya fue autorizado para salir. Caja debe cerrar la cuenta y cobrar. Una vez confirmado el pago, Admisión libera físicamente la habitación en el sistema.

---

## Automatización 1 — Alerta a Caja: Paciente en Egreso Físico

**Nombre en Jira:** `[NEXO] Alerta Caja — Paciente listo para cobro`
**Estado:** Nueva (pendiente de crear)

### Configuración Jira

| Campo | Valor |
|-------|-------|
| Trigger | Issue transitions to status |
| Status | `8. EGRESO FÍSICO (EJECUTIVAS)` |
| Action | Send email |

### Destinatario
- **Para:** Karen Cunto (correo registrado en Jira)

### Asunto del correo
```
[NEXO] Paciente listo para cobro — Hab. {{issue.fields.Habitación}}
```

### Cuerpo del correo
```
Hola Karen,

El siguiente paciente se encuentra en Egreso Físico y está listo para el cobro final:

  Paciente:        {{issue.fields.Nombre del paciente}}
  Habitación:      {{issue.fields.Habitación}}
  Tipo cobertura:  {{issue.fields.Tipo de cobertura}}
  Monto pendiente: {{issue.fields.Monto pendiente/vuelto}}
  Médico tratante: {{issue.fields.Médico tratante}}

Ver ticket: {{issue.url}}

Ariana (Caja) debe confirmar el cobro y mover el ticket a "9. Cama Liberada" una vez cancelada la cuenta.

— NEXO 360° Automatización
```

### Acción adicional — Comentario automático en el ticket
```
Comentario: "📋 Caja notificada para cobro final.
Hora de notificación: {{now | date('dd/MM/yyyy HH:mm')}}"
```

---

## Automatización 2 — Alerta demora Caja: Cobro sin confirmar +45 min

**Nombre en Jira:** `[NEXO] Alerta demora cobro Egreso Físico +45 min`
**Estado:** Nueva (pendiente de crear)

### Configuración Jira

| Campo | Valor |
|-------|-------|
| Trigger | Scheduled |
| Frecuencia | Every 15 minutes |
| JQL | `project = ALT AND status = "8. EGRESO FÍSICO (EJECUTIVAS)" AND updated <= -45m ORDER BY updated ASC` |
| Action | Send email |

### Destinatario
- **Para:** Karen Cunto (correo registrado en Jira)

### Asunto
```
[NEXO] ⚠️ Cobro pendiente +45 min — Hab. {{issue.fields.Habitación}}
```

### Cuerpo
```
ALERTA: El siguiente paciente lleva más de 45 minutos en Egreso Físico sin actualización:

  Paciente:        {{issue.fields.Nombre del paciente}}
  Habitación:      {{issue.fields.Habitación}}
  Tipo cobertura:  {{issue.fields.Tipo de cobertura}}
  Monto pendiente: {{issue.fields.Monto pendiente/vuelto}}
  Última edición:  {{issue.updated | date('dd/MM/yyyy HH:mm')}}

Ver ticket: {{issue.url}}

La habitación no puede liberarse hasta que Caja confirme el pago.

— NEXO 360° Automatización
```

> **Nota:** Si el monto pendiente es S/. 0.00 (paquete o cobertura total), ajustar el JQL con:
> `AND "Monto pendiente/vuelto" > 0`

---

## Automatización 3 — Confirmación de egreso: Cama disponible para Admisión

**Nombre en Jira:** `[NEXO] Cama liberada — Notificar Admisión y cerrar ticket`
**Estado:** Nueva (pendiente de crear)
*(Reemplaza o complementa la automatización existente "Notificación Cama Liberada → Admisión")*

### Configuración Jira

| Campo | Valor |
|-------|-------|
| Trigger | Issue transitions to status |
| Status | `9. CAMA LIBERADA (ADMISIÓN)` |
| Actions | 3 acciones encadenadas (ver abajo) |

### Acción 1 — Actualizar campo "Paciente en clínica"
```
Edit issue fields:
  Paciente en clínica = NO
```

### Acción 2 — Enviar notificación a Admisión

**Para:** Karen Cunto (correo registrado en Jira)

**Asunto:**
```
[NEXO] ✅ Habitación {{issue.fields.Habitación}} disponible — Paciente egresado
```

**Cuerpo:**
```
Hola Karen,

El siguiente paciente ha completado su egreso y la habitación está disponible:

  Paciente:    {{issue.fields.Nombre del paciente}}
  Habitación:  {{issue.fields.Habitación}}
  Cobertura:   {{issue.fields.Tipo de cobertura}}
  Médico:      {{issue.fields.Médico tratante}}

Acciones requeridas para Admisión (Valentino):
  [ ] Registrar desocupación de habitación en el sistema de admisión
  [ ] Confirmar disponibilidad para nuevo ingreso

Ver ticket: {{issue.url}}

— NEXO 360° Automatización
```

### Acción 3 — Comentario de cierre en el ticket
```
Comentario: "✅ Egreso completado.
Cuenta cancelada por Caja. Cama liberada para Admisión.
Cierre: {{now | date('dd/MM/yyyy HH:mm')}}"
```

---

## Resumen de las 3 automatizaciones

| # | Nombre | Trigger | Notifica a |
|---|--------|---------|------------|
| 1 | Alerta Caja — Paciente listo para cobro | Entra a columna 08 | Karen Cunto |
| 2 | Alerta demora cobro +45 min | Programada cada 15 min | Karen Cunto |
| 3 | Cama liberada — Notificar Admisión | Entra a columna 09 | Karen Cunto |

---

## Flujo completo con las automatizaciones activas

```
[07. Alta Aprobada]
        |
        | → AUTO EXISTENTE: Notificación Experiencia al Paciente (cobro en habitación)
        ↓
[08. Egreso Físico (Ejecutivas)]
        |
        | → AUTO 1: Email a Karen — paciente listo para cobro en Caja
        | → AUTO 2: Alerta a Karen +45 min si no hay movimiento
        |
        |  Ariana cobra y mueve ticket a columna 09 directamente
        ↓
[09. Cama Liberada (Admisión)]
        |
        | → AUTO 3: Actualiza "Paciente en clínica = NO"
        |           Email a Karen con habitación a liberar (para coordinar con Valentino)
        |           Comentario de cierre en el ticket
        ↓
     [FIN DEL FLUJO]
```

---

## Notas de implementación en Jira

1. Ir a **Project Settings → Automation → Create Rule**
2. Crear las 3 reglas en el orden indicado
3. En cada regla, usar **Send email → Specific email address** con el correo de Karen
4. Para la Automatización 2 (programada), verificar que el JQL filtra correctamente por tiempo con `updated <= -45m`
5. Ariana mueve el ticket de columna 08 → 09 directamente, sin requerir aprobación
6. No es necesario actualizar el campo `Monto pendiente/vuelto` antes de mover el ticket
7. Marcar como obsoleta la automatización existente "Notificación Cama Liberada → Admisión (on enter column 8)" si ya está cubierta por Automatización 3

---

## Pendiente definir con el equipo

- Mensaje exacto de WhatsApp al paciente confirmando su egreso (etapa 09, cuando Twilio esté activo)
