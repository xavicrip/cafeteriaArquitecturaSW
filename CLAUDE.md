# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Estado del repositorio

Proyecto en fase inicial: todavía no hay código, ni sistema de build, ni tests. Aún no existen comandos de build/lint/test; cuando se elija el stack y se haga el scaffolding, documéntalos aquí (incluido cómo ejecutar un solo test).

- `.specify/memory/constitution.md`: **constitución del proyecto** (principios I–VII no negociables). Prevalece sobre cualquier otro documento. Flujo Spec Kit: `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`; los specs viven en `specs/NNN-nombre/`.
- `promtp.md` (el nombre del archivo tiene esa errata): la especificación del producto. Es la fuente de verdad de los requisitos (RF-xx, RNF-xx), del stack y de la arquitectura objetivo (monolito modular en NestJS con outbox + cola, Redis, PostgreSQL); léela antes de diseñar o implementar.
- `ROADMAP.md`: plan por fases (0–7) con criterios de "listo", decisiones pendientes y riesgos. Consulta en qué fase está el proyecto antes de empezar trabajo nuevo.
- `.claude/skills/`: skills de diseño de UI instalados con `npx skills add` (registrados en `skills-lock.json`): `frontend-design` (Anthropic) y `design-taste-frontend` (taste-skill). Úsalos al construir o rediseñar la UI, sin sacrificar el requisito WCAG AAA (contraste ≥ 7:1, movimiento reducible).

## Producto: "Proyecto cafetería"

Aplicación web para gestionar una cafetería online.

Requisitos funcionales:
- Gestión de clientes.
- Compra de varios productos del menú, con un carrito de compras por cliente.
- Formas de pago: efectivo, transferencia y tarjeta de crédito.
- Menús dinámicos: se pueden modificar y actualizar en tiempo de ejecución, no son contenido fijo.
- Promociones y descuentos aplicados a distintos menús en ventanas de tiempo concretas (p. ej. festividades), es decir, reglas de precio acotadas por fechas.

Requisitos no funcionales que condicionan la arquitectura:
- Usabilidad, principios de UX y accesibilidad **WCAG AAA** (no solo AA).
- Alto volumen de pedidos concurrentes; el sistema debe escalar horizontalmente.
- Seguridad fuerte (incluye el manejo de pagos con tarjeta).
- Cumplimiento de la ley de protección de datos (datos personales de clientes).

## Commits y despliegue

- Commits **atómicos y reversibles** con Conventional Commits y el módulo como alcance (`feat(pedidos): ...`): un cambio lógico por commit, y cada commit debe compilar y pasar los tests por sí solo. Las migraciones siguen el patrón expand/contract (detalle en `promtp.md` §7).
- Frontend (`apps/web`) en **Vercel**, backend (API + workers) en **Render** vía `render.yaml`. Solo el pipeline de GitHub Actions despliega; el despliegue automático por Git está desactivado.
- El Quality Gate de **SonarQube** es obligatorio en cada PR; las métricas del pipeline y del equipo van a **Grafana** (§8–9).

## Restricción de stack

**No usar Python** en ninguna parte del stack (requisito explícito del usuario).

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->
