# Feature Spec: Session Tracking & Appointment Clearance (001)

## 1. Definición del Problema
Los profesionales de la salud y terapeutas suelen perder el control exacto de cuántas sesiones de un plan terapéutico (o derivación de obra social/prepaga) ha consumido un paciente. Al no tener visibilidad inmediata del saldo disponible al momento de atender o agendar:
1. Se sobrepasan las sesiones aprobadas sin acuerdo previo de pago particular.
2. Se pierde la oportunidad de evaluar un alta temprana o planificar a tiempo la extensión de sesiones antes de que el paciente agote su cupo.

## 2. Diseño de la Solución
Incorporar en la gestión de turnos un modelo de **Tratamiento Activo** (`TreatmentPlan`) que mantiene un conteo estricto de:
- `totalSessions`: Sesiones totales prescritas/acordadas.
- `usedSessions`: Turnos concretados.
- `remainingSessions`: Saldo disponible en tiempo real (`total - used`).

Cada vez que el profesional consulta la ficha del turno o marca la asistencia del paciente:
- El sistema muestra el estado visual del plan de sesiones.
- Se habilitan dos acciones directas: **"Dar de Alta"** (cierra el plan y cancela turnos futuros no consumidos) o **"Extender Tratamiento"** (añade sesiones extra con justificación registrada).

## 3. Criterios de Aceptación 

### CA-01: Descuento atómico de saldo al completar turno
**Scenario:** Paciente asiste a su turno
- **Given:** Un paciente tiene un plan activo con `remainingSessions = 3` y un turno en estado `PENDING`.
- **When:** El profesional marca el turno como `COMPLETED`.
- **Then:** `usedSessions` se incrementa en 1, `remainingSessions` pasa a ser 2, y el cambio se persiste de forma transaccional.

### CA-02: Alerta visual de umbral crítico de sesiones restantes
**Scenario:** El plan del paciente está por finalizar
- **Given:** El profesional abre la vista del turno o la ficha del paciente.
- **When:** `remainingSessions` es menor o igual a 1 y mayor a 0.
- **Then:** El sistema muestra una alerta de estado `CRITICAL_BALANCE` recomendando al profesional evaluar si corresponde otorgar el alta o renovar la prescripción antes del próximo agendamiento.

### CA-03: Cierre anticipado de tratamiento (Alta Médica)
**Scenario:** Profesional otorga el alta antes de agotar las sesiones
- **Given:** Un paciente cuenta con `remainingSessions > 0` y tiene 2 turnos futuros agendados en estado `SCHEDULED`.
- **When:** El profesional selecciona la acción "Otorgar Alta Médica" registrando el motivo clínico.
- **Then:** El plan pasa a estado `DISCHARGED`, los turnos futuros pasan automáticamente a estado `CANCELLED_BY_PROVIDER` con motivo "Alta médica", y se desactiva cualquier recordatorio programado para dichos turnos.

### CA-04: Bloqueo de sobrecupo sin confirmación explícita
**Scenario:** Intento de agendar con saldo agotado
- **Given:** Un paciente con plan activo pero `remainingSessions == 0`.
- **When:** El profesional intenta registrar un nuevo turno dentro de ese plan.
- **Then:** El sistema bloquea la creación por defecto arrojando un error `422 Unprocessable Entity` con código `QUOTA_EXHAUSTED`, requiriendo explícitamente una extensión del plan (`extendSessions`) o un cambio de modalidad a sesión particular fuera de plan.