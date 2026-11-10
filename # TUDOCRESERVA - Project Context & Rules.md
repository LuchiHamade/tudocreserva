# TUDOCRESERVA - Project Context & Rules

## 1. Propósito y Dominio
Sistema web para la gestión de turnos clínicos, seguimiento de planes de tratamiento terapéutico/médico (paquetes de sesiones) y reducción del ausentismo mediante recordatorios automatizados.

## 2. Stack Tecnológico & Arquitectura
- **Backend:** Node.js (TypeScript) / Express 
- **Frontend:** SPA con TypeScript, React y  CSS.
- **Base de Datos:** PostgreSQL.
- **Seguridad & Cumplimiento:**
  - Hasheo de credenciales con Argon2/Bcrypt.
  - Comunicaciones en tránsito cifradas bajo TLS 1.3.
  - Privacidad estricta de datos de salud (Ley de Protección de Datos Personales): los registros médicos y el conteo de sesiones están estrictamente aislados por profesional (`tenant/provider isolation`).

## 3. Patrones de Diseño & Buenas Prácticas
- **Arquitectura en Capas:** Controller -> Service -> Repository.
- **DTOs & Serialización:** Jamás exponer entidades directas de la base de datos hacia el frontend. Utilizar DTOs con tipado estricto para request y response, omitiendo metadatos internos y campos sensibles.
- **Idempotencia:** Toda operación que modifique contadores de saldo o dispare transacciones/notificaciones debe admitir una cabecera `Idempotency-Key` (UUIDv4) o validar el estado previo para evitar duplicaciones.
- **Manejo de Errores:** Errores canónicos tipados (`NotFoundError`, `ConflictError`, `BusinessRuleValidationError`) capturados por un middleware global.

## 4. Reglas del Asistente / Agente
- Respeta estrictamente los contratos de interfaz definidos en las especificaciones (`.spec.md`).
- Si una especificación resulta ambigua o incompleta durante la codificación, no asumas la solución: detén el código y solicita ajustar la spec.