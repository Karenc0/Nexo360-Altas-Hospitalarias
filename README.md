# NEXO 360° — Plataforma de Gestión de Altas Hospitalarias
**Clínica San Felipe | Área de Altas Hospitalarias**

---

## Descripción

**NEXO 360°** es una plataforma digital de gestión de altas hospitalarias implementada sobre **Jira Software (Free Plan)** para la Clínica San Felipe. El sistema digitaliza y automatiza el flujo de alta médica desde la orden del médico hasta la liberación de la cama, eliminando la coordinación informal por WhatsApp entre 7 departamentos.

**Costo de implementación: USD 0**

---

## Problema que resuelve

| Antes | Después |
|-------|---------|
| Coordinación por WhatsApp (informal, sin trazabilidad) | Flujo digital con 9 etapas visibles en tiempo real |
| Sin registro de tiempos | KPIs y métricas automáticas |
| Sin alertas de demora | 11 automatizaciones activas |
| Sin visibilidad para jefatura | Dashboard de control con gadgets |

---

## Arquitectura del Sistema

### Flujo Kanban — 9 Etapas

```
Alta Medica → Farmacia → Enfermeria → Aseguradora → Medicos Auditores
     → Alta Rapida → Alta Aprobada → Egreso Fisico → Cama Liberada
```

| # | Etapa | Area Responsable |
|---|-------|-----------------|
| 01 | Alta Medica | Medico / Hospitalizacion |
| 02 | Farmacia | Farmacia |
| 03 | Enfermeria (Cierre HC) | Enfermeria |
| 04 | Aseguradora / Presupuesto | Admision |
| 05 | Medicos Auditores | Auditoria Medica |
| 06 | Alta Rapida | Altas |
| 07 | Alta Aprobada | Altas |
| 08 | Egreso Fisico (Ejecutivas) | Caja / Altas |
| 09 | Cama Liberada (Admision) | Admision |

### 10 Campos Personalizados

- Habitacion
- Nombre del paciente
- Tipo de cobertura (Particular / Seguro EPS / Convenio / Paquete prepagado)
- Medico tratante
- Hora de orden de alta
- Estado de carta (Pendiente / Parcial / Al 100%)
- Monto pendiente / vuelto
- Monto pago a cuenta
- Liquidacion enviada
- Tipo de alta (Estandar / Rapida / Compleja)

### 11 Automatizaciones Activas

| # | Nombre | Trigger |
|---|--------|---------|
| 1 | Alerta Alta Rapida +20 min | Programado |
| 2 | Alerta diaria — Cuentas sin movimiento +3 dias | Programado 8am |
| 3 | Alerta ticket sin movimiento +2h | Programado |
| 4 | Notificacion: Alta lista para Administrativo | Transicion |
| 5 | Notificacion automatica a Farmacia | Transicion col. 2 |
| 6 | Notificacion Cama Liberada a Admision | Transicion col. 8 |
| 7 | Notificacion Experiencia al Paciente | Transicion col. 7 |
| 8 | Orden de limpieza — Egreso Fisico | Transicion col. 8 |
| 9 | Alerta Caja — Paciente listo para cobro | Transicion col. 8 |
| 10 | Alerta demora cobro — Egreso Fisico +45 min | Programado c/15 min |
| 11 | Cama Liberada — Notificar Admision | Transicion col. 9 |

---

## SLA por Tipo de Cuenta

| Tipo | SLA Maximo |
|------|-----------|
| Alerta financiera / monto alto | 24 horas |
| Particular / Corte de cuenta | 48 horas |
| Seguro con carta parcial | 72 horas |
| Paquete prepagado / 100% CG | 5 dias |

---

## Filtros JQL Guardados

```sql
-- Altas Rapidas en curso
project = ALT AND status = "6. ALTA RAPIDA" ORDER BY created DESC

-- Altas en proceso ahora mismo
project = ALT AND status NOT IN ("9. CAMA LIBERADA (Admision)") ORDER BY created ASC

-- Cuentas sin movimiento +3 dias
project = ALT AND status NOT IN ("9. CAMA LIBERADA (Admision)") AND updated <= -3d ORDER BY updated ASC

-- Alerta financiera - Prioridad critica
project = ALT AND status NOT IN ("9. CAMA LIBERADA (Admision)") AND priority = Highest ORDER BY created ASC

-- Cuentas Abiertas Post-Egreso
project = ALT AND "Hora de orden de alta" <= now() AND status NOT IN ("9. CAMA LIBERADA (Admision)") ORDER BY "Hora de orden de alta" ASC
```

---

## Contenido del Repositorio

```
ProyectoNexo/
├── README.md                          # Este archivo
├── CLAUDE.md                          # Arquitectura tecnica completa
├── NEXO360_Documentacion_Completa.docx # Documentacion master del proyecto
├── Auditoria_Tecnica_NEXO360.pdf      # Informe de auditoria pre-UAT
└── automatizaciones/
    └── flujo_caja_admision.md         # Spec tecnica: flujo Caja -> Admision
```

---

## Equipo

| Nombre | Rol |
|--------|-----|
| Karen Cunto | Project Lead — Altas Hospitalarias |
| Jocelyn Quispe | Enfermeria / Obstetricia |
| Ariana Ibarra | Caja / Tesoreria |
| Valentino Esteban | Admision |

---

## Integracion WhatsApp (Pendiente)

- **Herramienta:** Twilio (~USD 0.05/mensaje)
- **Estado:** Pendiente aprobacion Meta (3-7 dias habiles)
- **Etapas con notificacion:** 3, 4, 7 y 8

---

## Stack Tecnologico

- **Plataforma:** Jira Software (Free Plan)
- **Proyecto:** `ALT` — Kanban
- **Automatizacion:** Jira Automation Rules
- **Notificaciones:** Email + WhatsApp (pendiente)
- **Costo mensual:** USD 0 (hasta 10 usuarios)

---

*Proyecto desarrollado para el concurso de innovacion hospitalaria — Clinica San Felipe, 2026.*
