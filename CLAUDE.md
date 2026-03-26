# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**NEXO 360°** is a digital hospital discharge management platform built on **Jira Software** for Clínica San Felipe — Área de Altas Hospitalarias. This is not a traditional software project; it is a Jira workflow configuration, process documentation, and integration design project.

The system digitizes a 9-stage discharge workflow, automates notifications across 7 hospital departments, and eliminates informal WhatsApp coordination. Implementation cost: zero (Jira free plan up to 10 users).

## Repository Contents

Currently the repository contains:
- `NEXO360_Documentacion_Completa.docx` — The single source of truth for the entire project (Jira config, automations, checklists, KPIs, pending tasks, contest submission answers)

## Project Architecture

### Jira Project
- **Project name:** Proyecto NEXO - Clínica San Felipe
- **Key:** `ALT`
- **Board type:** Kanban
- **Plan:** Free (up to 10 users → ~USD 8/user/month if it grows)

### 9-Stage Kanban Workflow
| # | Column | Responsible Area |
|---|--------|-----------------|
| 01 | Alta Médica | Médico / Hospitalización |
| 02 | Farmacia | Farmacia |
| 03 | Enfermería (Cierre HC) | Enfermería / Jocelyn |
| 04 | Aseguradora / Presupuesto | Aseguradoras / Admisión |
| 05 | Médicos Auditores | Médicos Auditores |
| 06 | Alta Rápida | Altas |
| 07 | Alta Aprobada | Altas |
| 08 | Egreso Físico (Ejecutivas) | Altas |
| 09 | Cama Liberada (Admisión) | Admisión |

### 10 Custom Fields in Jira
Habitación, Nombre del paciente, Tipo de cobertura (Particular/Rímac/Pacífico/EPS/Paquete), Médico tratante, Hora de orden de alta, Estado de carta (Pendiente/Parcial/Al 100%), Monto pendiente/vuelto, Liquidación enviada, Paciente en clínica, Observaciones.

### 7 Active Automations
- Alerta Alta Rápida +20 min
- Alerta diaria — cuentas sin movimiento +3 días (8am daily)
- Alerta ticket sin movimiento +2h
- Notificación: Alta lista para Administrativo
- Notificación automática a Farmacia (on enter column 2)
- Notificación Cama Liberada → Admisión (on enter column 8)
- Notificación Experiencia al Paciente — Cobro en Habitación (on enter column 7)

### 3 Pending Automations — Flujo Final Caja → Admisión
*(Spec: `automatizaciones/flujo_caja_admision.md`)*
- Alerta Caja: Paciente listo para cobro (on enter column 8)
- Alerta demora cobro Egreso Físico +45 min (scheduled every 15 min)
- Cama liberada: Notificar Admisión + actualizar campos (on enter column 9)

### 4 Saved Filters (JQL)
- **Altas Rápidas en curso:** `project = ALT AND status = "6. ALTA RÁPIDA" ORDER BY created DESC`
- **Altas en proceso ahora mismo:** `project = ALT AND status NOT IN ("8. EGRESO FÍSICO (EJECUTIVAS)") ORDER BY created ASC`
- **Cuentas sin movimiento +3 días:** `project = ALT AND status NOT IN ("8. EGRESO FÍSICO (EJECUTIVAS)") AND updated <= -3d ORDER BY updated ASC`
- **Alerta financiera - Prioridad crítica:** `project = ALT AND status NOT IN ("8. EGRESO FÍSICO (EJECUTIVAS)") AND priority = Highest ORDER BY created ASC`

### SLAs by Account Type
| Type | Max SLA |
|------|---------|
| Financial alert / high amount | 24 hours |
| Particular / Corte de cuenta | 48 hours |
| Insurance with partial letter | 72 hours |
| Prepaid package / 100% CG | 5 days |

## Planned WhatsApp Integration
- **Status:** Pending coordination with Sistemas
- **Tool:** Twilio (~USD 0.05/message), requires Meta approval (3–7 business days)
- Sends automatic updates to patients at stages 3, 4, 7, and 8

## Team
| Name | Role |
|------|------|
| Karen Cunto | Project Lead — Altas Hospitalarias |
| Jocelyn Quispe | Enfermería / Obstetricia |
| Ariana Ibarra | Caja / Tesorería |
| Valentino Esteban | Admisión |

## Pending Tasks
- Complete Caja and Admisión checklists in real tickets
- Block 6 — Invite team and assign roles in Jira
- Block 10 — Reports and metrics dashboard in Jira
- Coordinate with Sistemas to start Meta/Twilio approval process
- Define exact WhatsApp messages per stage with the team
- Confirm contest deadline and submit final form
