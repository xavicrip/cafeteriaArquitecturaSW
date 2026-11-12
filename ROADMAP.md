# Roadmap — Proyecto Cafetería

Plan de implementación derivado de [`promtp.md`](promtp.md). Las referencias `RF-xx` / `RNF-xx` apuntan a sus requisitos.

> **Supuestos de planificación:** sprints de 2 semanas y un equipo pequeño (2–4 personas). Las duraciones son orientativas; ajústalas al equipo real.

## Resumen

| Fase | Objetivo | Requisitos | Duración |
|---|---|---|---|
| 0. Fundaciones | Repo, flujo de Git, pipeline CI/CD con SonarQube, Vercel + Render, Grafana y bases de seguridad, datos y accesibilidad | RNF-01, 03, 04, 06 | 2 sprints |
| 1. Identidad y clientes | Registro, login, roles, perfil y derechos LOPDP | RF-01 | 1 sprint |
| 2. Catálogo dinámico | Menús versionados, disponibilidad y caché | RF-02 | 1–2 sprints |
| 3. Carrito y promociones | Carrito persistente y motor de precios | RF-03, RF-06 | 2 sprints |
| 4. Pedidos y pagos | Checkout, outbox + cola, pasarela, tiempo real | RF-04, RF-05 | 2–3 sprints |
| **Hito: MVP** | Un cliente compra y paga de punta a punta en staging | — | — |
| 5. Administración y reportes | Panel de administración, reportes y auditoría | RF-07 | 1–2 sprints |
| 6. Endurecimiento | Carga, seguridad, AAA, LOPDP, observabilidad y producción | RNF-01…06 | 2 sprints |
| **Hito: lanzamiento** | Producción con datos reales | — | — |
| 7. Operación y evolución | Mejora continua y extracción de servicios si la carga lo exige | RNF-02, 05 | continuo |

Total estimado hasta el lanzamiento: **~12–15 sprints (6–7,5 meses)**.

---

## Fase 0 — Fundaciones

**Objetivo:** que cualquier persona del equipo pueda clonar, levantar todo en local y que cada PR pase por el pipeline completo hasta un despliegue de preview.

- Monorepo TypeScript (pnpm + Turborepo): `apps/web` (Next.js), `apps/api` (NestJS), `apps/worker`, `packages/*` (tipos y contratos compartidos, UI).
- Estructura de módulos de dominio vacía en NestJS con reglas de dependencia entre módulos verificadas en CI (p. ej. `eslint-plugin-boundaries`).
- `docker compose` local: PostgreSQL, Redis, RabbitMQ y un servidor de correo de pruebas.
- **Flujo de Git** (`promtp.md` §7): Husky + commitlint + lint-staged, protección de `main`, merge con rebase e historial lineal, release-please.
- **Pipeline de CI/CD** (`promtp.md` §8) en GitHub Actions: validación commit por commit, lint, typecheck, tests unitarios y de integración, escaneos de seguridad, build y preview.
- **SonarQube** conectado al repo, con el Quality Gate de §8.4 como check obligatorio del PR.
- **Despliegue:** proyecto de Vercel para `apps/web` y Blueprint `render.yaml` (API, worker, Postgres, Key Value, RabbitMQ) con entornos `staging` y `prod`; despliegue automático por Git desactivado.
- **Grafana:** data sources de GitHub, del pipeline y de SonarQube, y primeros dashboards (DORA y salud del pipeline).
- **Diseño:** sistema de diseño accesible (tokens con contraste ≥ 7:1, tipografía y componentes base) usando el skill `frontend-design`.
- **Seguridad y datos:** modelo de amenazas inicial e inventario de datos personales (qué se guarda, para qué y cuánto tiempo).
- Registros de decisiones (ADR) para el stack y la arquitectura.

**Listo cuando:** un PR con un endpoint `/health` y una página de inicio pasa el Quality Gate, se despliega en preview y, tras el merge, llega solo a `staging`. Ese PR también debe aparecer en el dashboard DORA de Grafana, y el `git revert` de su commit debe desplegarse igual de limpio.

## Fase 1 — Identidad y clientes (RF-01)

- Registro, login, recuperación de contraseña (Argon2id) y sesiones en Redis.
- Roles `cliente`, `personal` y `admin`; MFA obligatorio para personal y administradores.
- Perfil del cliente con PII cifrada por columna en el esquema `customers`.
- Consentimientos y derechos LOPDP: ver, editar, exportar (JSON) y eliminar la cuenta.
- Rate limiting en login y registro.

**Listo cuando:** un cliente gestiona su cuenta y sus datos solo con teclado y lector de pantalla, y los tests de autorización por rol pasan.

## Fase 2 — Catálogo dinámico (RF-02)

- CRUD de menús, categorías y productos (alérgenos, imagen con texto alternativo).
- Versionado de menús, publicar o retirar, disponibilidad por horario y por existencias.
- Lectura del menú desde Redis + CDN de Vercel; cada publicación emite un evento que invalida la caché.
- Vista pública del menú (SSR, mobile-first, AAA).

**Listo cuando:** un cambio publicado por el admin aparece en la tienda en segundos sin despliegue, y la lectura del menú cumple p95 < 300 ms en una prueba de carga básica.

## Fase 3 — Carrito y promociones (RF-03, RF-06)

- Carrito por cliente en Redis con respaldo en PostgreSQL; se sincroniza entre dispositivos.
- Motor de precios del módulo Promociones: tipos (porcentaje, monto fijo, 2x1, combo), alcance, ventana de vigencia, cupones, prioridad y acumulabilidad.
- Tests exhaustivos del motor: casos límite de fechas y zonas horarias, combinaciones de promociones y redondeo monetario (aritmética con enteros o decimales, nunca coma flotante).
- El carrito muestra el precio final calculado en el servidor y avisa de cambios de precio o de disponibilidad.

**Listo cuando:** una promoción programada para una festividad se activa y desactiva sola en su ventana, y el precio del carrito coincide siempre con el del servidor.

## Fase 4 — Pedidos y pagos (RF-04, RF-05)

Fase de mayor riesgo técnico: concentra la concurrencia y el dinero.

- Checkout con `Idempotency-Key`: valida disponibilidad, recalcula el precio y crea el pedido con una copia inmutable de precios, más el evento outbox, en una sola transacción.
- Publicador del outbox → RabbitMQ; workers idempotentes, cola de mensajes fallidos (DLQ) y reintentos con backoff.
- Máquina de estados del pedido y cola de cocina para el personal.
- **Tarjeta:** integración con la pasarela en sandbox (campos alojados o redirección), webhooks firmados y deduplicados, circuit breaker.
- **Transferencia y efectivo:** confirmación manual por el personal, con auditoría.
- Estado del pedido en tiempo real (WebSocket/SSE + Redis pub/sub).
- Notificaciones por correo.

**Listo cuando (hito MVP):** en staging, un cliente compra con cada forma de pago y sigue su pedido hasta `entregado`. Una prueba que mata workers y repite webhooks no pierde ni duplica pedidos ni cobros.

## Fase 5 — Administración y reportes (RF-07)

- Panel de administración: menús, promociones, pedidos y usuarios del personal.
- Reportes de ventas, productos más vendidos y uso de promociones, leídos desde la réplica.
- Auditoría inmutable de cambios en precios, menús, promociones y pagos.

**Listo cuando:** el administrador opera la cafetería sin acceso directo a la base de datos.

## Fase 6 — Endurecimiento para producción

- **Carga (RNF-02):** pruebas k6 con ≥ 1 000 pedidos/min sostenidos y picos; ajuste del autoescalado de Render, PgBouncer, índices y particionado de `orders` si hace falta.
- **Seguridad (RNF-03):** revisión con OWASP ASVS L2, pentest externo, WAF, gestor de secretos y rotación de credenciales.
- **Accesibilidad (RNF-01):** auditoría WCAG 2.2 AAA con usuarios reales y lectores de pantalla; corrección de hallazgos.
- **Protección de datos (RNF-04):** revisión legal LOPDP, política de retención y anonimización automática, registro de actividades de tratamiento.
- **Operación (RNF-05, 06):** dashboards y alertas en Grafana (OpenTelemetry, Prometheus, Loki, Tempo), SLO de 99,9 %, runbooks, backups de Render Postgres con prueba de restauración, release a `prod` con aprobación manual y rollback automático probado en Vercel y Render.

**Listo cuando (hito lanzamiento):** se cumplen las metas de carga y SLO en staging, no quedan hallazgos críticos ni altos de seguridad o accesibilidad, y la restauración desde backup está probada.

## Fase 7 — Operación y evolución

- Pruebas de carga antes de cada temporada de festividades.
- Revisar métricas trimestralmente: si Pedidos o Pagos saturan el monolito, extraerlos como servicios (los límites de módulo y los eventos ya lo permiten).
- Evaluar Kafka si el volumen de eventos supera lo que RabbitMQ maneja con holgura.

---

## Transversal a todas las fases

- Cada historia incluye criterios de accesibilidad, seguridad y datos personales en su definición de "terminado".
- Los tests de integración y E2E de cada fase se suman al pipeline, sin dejarlos para el final.
- ADR para cada decisión de arquitectura relevante.

## Decisiones pendientes

| Decisión | Necesaria antes de |
|---|---|
| Plan de Render (autoescalado, preview environments y pre-deploy requieren plan de pago) y plan de Vercel | Fase 0 |
| SonarQube Cloud o servidor propio; Grafana Cloud o instancia propia | Fase 0 |
| RabbitMQ autogestionado en Render o CloudAMQP | Fase 0 |
| Confirmar jurisdicción (LOPDP Ecuador) | Fase 1 |
| Validar las metas de carga y latencia con el negocio | Fase 2 |
| Pasarela de pago y sus requisitos de integración | Fase 4 |
| Método de conciliación de transferencias (manual o API bancaria) | Fase 4 |

## Riesgos principales

| Riesgo | Mitigación |
|---|---|
| Pagos duplicados o perdidos | Idempotencia, outbox, deduplicación de webhooks y pruebas de caos en la fase 4 |
| Errores de precio por combinaciones de promociones | Motor de precios aislado, con tests exhaustivos y copia inmutable en el pedido |
| AAA inalcanzable en algún contenido | Planificar desde el sistema de diseño; documentar excepciones justificadas |
| Picos en festividades superiores a lo previsto | Cola que absorbe picos, autoescalado y pruebas de carga previas a cada temporada |
| Retraso de la pasarela o de la revisión legal | Iniciar ambos trámites en la fase 0 |
