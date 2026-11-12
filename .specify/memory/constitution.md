<!--
Sync Impact Report
- Versión: plantilla sin versión → 1.0.0 (adopción inicial)
- Principios definidos (plantilla → nuevo):
  - [PRINCIPLE_1_NAME] → I. Accesibilidad AAA y experiencia de usuario
  - [PRINCIPLE_2_NAME] → II. Seguridad por diseño
  - [PRINCIPLE_3_NAME] → III. Protección de datos personales (LOPDP)
  - [PRINCIPLE_4_NAME] → IV. Escalabilidad y consistencia de pedidos
  - [PRINCIPLE_5_NAME] → V. Monolito modular con límites de dominio
  - Añadido → VI. Calidad verificada (NO NEGOCIABLE)
  - Añadido → VII. Commits atómicos y reversibles (NO NEGOCIABLE)
- Secciones añadidas: Restricciones tecnológicas y de plataforma; Flujo de desarrollo,
  entrega y métricas; Gobernanza.
- Secciones eliminadas: ninguna.
- Plantillas:
  - ✅ .specify/templates/plan-template.md: Constitution Check con las puertas I–VII.
  - ✅ .specify/templates/tasks-template.md: los tests pasan a ser obligatorios (principio VI);
    las rutas de ejemplo usan TypeScript.
  - ✅ .specify/templates/spec-template.md: revisada; sus secciones de requisitos y criterios
    de éxito ya cubren lo necesario, sin cambios.
  - ✅ CLAUDE.md: referencia a la constitución.
- TODO diferidos: ninguno.
-->

# Constitución del Proyecto Cafetería

Fuentes: `promtp.md` (especificación del producto y la arquitectura), `ROADMAP.md` (fases) y
`CLAUDE.md` (guía del agente). Si alguno de ellos contradice esta constitución, prevalece la
constitución y el otro documento debe corregirse.

## Core Principles

### I. Accesibilidad AAA y experiencia de usuario

- Toda interfaz DEBE cumplir **WCAG 2.2 nivel AAA**: contraste de texto ≥ 7:1, operación completa
  por teclado con foco visible, objetivos táctiles ≥ 44×44 px, sin límites de tiempo en el
  checkout, movimiento reducible (`prefers-reduced-motion`), lenguaje claro y mensajes de error
  que indiquen cómo corregirlos.
- Cuando un contenido concreto no pueda cumplir AAA, la excepción DEBE documentarse en el spec
  con su justificación y cumplir como mínimo AA.
- El diseño es mobile-first y responsive. Ningún skill de diseño ni preferencia estética
  justifica incumplir este principio.
- Verificación: axe sin violaciones en el pipeline y pruebas manuales con lector de pantalla
  (NVDA o VoiceOver) en los flujos críticos antes de cada release.

**Razón:** la accesibilidad AAA es un requisito explícito del producto, y añadirla al final
cuesta mucho más que diseñar con ella desde el principio.

### II. Seguridad por diseño

- El sistema DEBE alinearse con **OWASP ASVS nivel 2** y no presentar riesgos abiertos del
  OWASP Top 10.
- Los datos de tarjeta NUNCA pasan por nuestros servidores: el pago con tarjeta se hace con los
  campos alojados o la redirección de la pasarela (tokenización), dejando el sistema fuera del
  alcance de PCI DSS.
- Contraseñas con Argon2id; **MFA obligatorio** para personal y administradores; control de
  acceso por roles comprobado en el servidor en cada operación.
- Los secretos NUNCA se guardan en el repositorio: viven en las variables de entorno cifradas de
  Vercel, Render y GitHub Environments. El pipeline bloquea cualquier secreto detectado.
- Las vulnerabilidades altas o críticas en dependencias o imágenes bloquean el merge.

**Razón:** el sistema maneja dinero y datos personales; un fallo de seguridad compromete el
negocio y a los clientes.

### III. Protección de datos personales (LOPDP)

- El tratamiento de datos DEBE cumplir la **Ley Orgánica de Protección de Datos Personales del
  Ecuador**.
- Minimización: solo se recoge lo necesario para la finalidad declarada, y cada dato personal
  nuevo DEBE añadirse al inventario de datos con su finalidad y plazo de retención.
- Consentimiento explícito y revocable para finalidades no esenciales (p. ej. marketing).
- El cliente DEBE poder ejercer sus derechos de acceso, rectificación, eliminación, oposición y
  portabilidad desde la propia aplicación.
- La PII se cifra en tránsito y en reposo (cifrado por columna) y solo la gestiona el módulo
  Clientes; el resto de módulos usa el identificador del cliente.
- Eliminar una cuenta anonimiza los pedidos en lugar de borrarlos, para conservar la contabilidad.
- Las métricas de actividad del equipo de desarrollo también son datos personales y están
  sujetas a este principio.

**Razón:** cumplir la ley es obligatorio, y la minimización reduce el impacto de cualquier
incidente.

### IV. Escalabilidad y consistencia de pedidos

- Los servicios (frontend, API y workers) DEBEN ser **sin estado**: sesión, carrito y caché viven
  en Redis (Key Value) o en Postgres, para poder escalar horizontalmente.
- Toda operación que cambie el estado de un pedido o un pago y deba notificar a otros módulos
  DEBE escribir su evento en la misma transacción (**patrón outbox**) y procesarse en segundo
  plano mediante la cola.
- Las operaciones de creación de pedidos y de confirmación de pagos DEBEN ser **idempotentes**
  (`Idempotency-Key`, deduplicación de webhooks, consumidores tolerantes a entregas repetidas).
- El precio final se calcula SIEMPRE en el servidor, y el pedido guarda una **copia inmutable**
  de precios, descuentos y descripción vigentes al comprar.
- Ningún pedido ni cobro puede perderse ni duplicarse ante fallos o reintentos; los specs de
  pedidos y pagos DEBEN incluir escenarios de fallo que lo demuestren.
- Metas de referencia: ≥ 1 000 pedidos/min en pico, p95 < 300 ms en lectura del menú y < 800 ms
  al confirmar un pedido. Las metas se validan con pruebas de carga.

**Razón:** las festividades generan picos de demanda; perder o duplicar un pedido o un cobro es
el fallo más grave posible del sistema.

### V. Monolito modular con límites de dominio

- El backend es un **monolito modular** en NestJS. Módulos de dominio: Identidad, Clientes,
  Catálogo, Promociones, Carrito, Pedidos, Pagos, Notificaciones y Reportes.
- Cada módulo es dueño de su esquema de base de datos. Ningún módulo lee ni escribe las tablas
  de otro: se comunican solo mediante interfaces públicas o eventos.
- Los límites entre módulos se verifican automáticamente en el pipeline (reglas de lint).
- Extraer un módulo como servicio independiente requiere un ADR justificado con métricas de
  carga.
- Toda dependencia o abstracción nueva DEBE justificarse; ante la duda, la opción más simple.

**Razón:** límites claros permiten escalar el equipo y extraer servicios después, sin pagar
desde el principio la complejidad de los microservicios.

### VI. Calidad verificada (NO NEGOCIABLE)

- Todo cambio de comportamiento DEBE incluir tests automatizados: unitarios para la lógica,
  integración (Testcontainers con Postgres, Redis y RabbitMQ reales) para persistencia y
  mensajería, y E2E (Playwright) para los flujos críticos del usuario.
- La lógica de precios, promociones, pedidos y pagos DEBE escribirse con TDD: test que falla →
  implementación → refactor.
- El **Quality Gate de SonarQube** es obligatorio en cada PR, aplicado al código nuevo:
  - cobertura ≥ 80 % (≥ 95 % en los módulos `promociones`, `pedidos` y `pagos`);
  - 0 bugs nuevos y 0 vulnerabilidades nuevas;
  - 100 % de los security hotspots revisados;
  - duplicación < 3 %;
  - calificación A en mantenibilidad, fiabilidad y seguridad.
- Un PR que no pasa el Quality Gate no se integra. Bajar un umbral requiere una enmienda de esta
  constitución.
- Los montos se representan con enteros (centavos) o decimales exactos, nunca en coma flotante.

**Razón:** los tests y el Quality Gate son lo que permite cambiar el sistema rápido sin
romperlo, y los errores en dinero no se toleran.

### VII. Commits atómicos y reversibles (NO NEGOCIABLE)

- **Atómico:** cada commit contiene un único cambio lógico; no se mezclan refactor,
  funcionalidad y formato.
- **Verde:** cada commit compila y pasa sus tests por sí solo; el pipeline lo comprueba commit a
  commit.
- **Mensaje:** Conventional Commits con el módulo como alcance (`feat(pedidos): ...`); los
  cambios incompatibles se marcan con `!` o `BREAKING CHANGE:`. Lo validan commitlint (Husky en
  local) y el pipeline.
- **Reversible:** `git revert` de cualquier commit de `main` DEBE producir un estado
  desplegable. Para ello:
  - las migraciones siguen el patrón **expand/contract** y son compatibles con la versión
    anterior de la aplicación;
  - una migración ya aplicada en producción se deshace con una migración nueva, no con
    `git revert`;
  - las funcionalidades incompletas se integran detrás de **feature flags**.
- Merge a `main` solo por PR, con **rebase** e historial lineal; los arreglos se integran antes
  con `--fixup` y `--autosquash`.

**Razón:** los commits atómicos y reversibles hacen que cualquier cambio problemático se pueda
aislar y deshacer en minutos, y que el historial explique por qué cambió el sistema.

## Restricciones tecnológicas y de plataforma

- **Lenguaje:** TypeScript en frontend, backend y workers. **No se usa Python** en ninguna parte
  del stack de la aplicación. (Las herramientas de desarrollo, como la CLI de Spec Kit, quedan
  fuera de esta restricción).
- **Frontend:** Next.js desplegado en **Vercel**.
- **Backend:** API NestJS y workers desplegados en **Render**, definidos en el Blueprint
  `render.yaml`.
- **Datos y mensajería:** Render Postgres, Render Key Value (Redis) y RabbitMQ. Postgres, Key
  Value, RabbitMQ y los workers solo son accesibles desde la red privada de Render.
- **Despliegue:** **solo el pipeline despliega**. El despliegue automático por Git está
  desactivado en Vercel y Render, y nadie despliega desde su máquina.
- Vercel y Render se ubican en la misma región.
- Cambiar una tecnología de esta sección requiere un ADR y una enmienda de la constitución.

## Flujo de desarrollo, entrega y métricas

- **Spec-driven:** cada funcionalidad pasa por spec → plan → tasks (Spec Kit) antes de
  implementarse. El spec describe el qué y el porqué; el plan, el cómo.
- **Ramas:** trunk-based, con ramas cortas (< 2 días) desde `main`. `main` está protegida: PR
  obligatorio, al menos 1 aprobación y todos los checks en verde.
- **Pipeline CI/CD (GitHub Actions):**
  - En cada PR: validación de commits, lint y typecheck, tests unitarios y de integración,
    SonarQube, escaneos de seguridad, build, despliegue de preview, E2E y axe.
  - En cada merge a `main`: build de imágenes y despliegue automático a `staging` con smoke tests.
  - Release a producción: **aprobación manual**; se despliega primero el backend y después el
    frontend.
- **Rollback:** si los smoke tests de producción fallan, el pipeline revierte automáticamente en
  Vercel y Render. Después se hace `git revert` para que `main` refleje lo que está en
  producción.
- **Métricas del equipo en Grafana:** DORA, flujo de trabajo, calidad, salud del pipeline y
  aporte del equipo, con datos de GitHub, del pipeline y de SonarQube.
  - Las metas y alertas se fijan por **equipo**.
  - Las métricas individuales sirven para hacer visible el trabajo (incluidas las revisiones y
    los tests) y nunca para hacer rankings ni evaluaciones automáticas.
  - El acceso al dashboard individual está restringido.
- **Revisión de código:** el revisor verifica el cumplimiento de esta constitución, en especial
  los principios I, III, IV y VII, que no se pueden comprobar del todo de forma automática.

## Governance

- Esta constitución prevalece sobre cualquier otra práctica o documento del proyecto.
- **Enmiendas:** por PR que modifique este archivo, con la justificación, el impacto en specs,
  planes y plantillas, y el plan de migración si afecta al código existente. Requiere la
  aprobación de al menos 2 miembros del equipo.
- **Versionado semántico de la constitución:**
  - MAJOR: se elimina o redefine un principio de forma incompatible.
  - MINOR: se añade un principio o sección, o se amplía de forma material una regla.
  - PATCH: aclaraciones y redacción sin cambio de significado.
- Cada enmienda actualiza el Sync Impact Report, la versión y la fecha de última enmienda, y
  propaga los cambios a `.specify/templates/` y a `CLAUDE.md`.
- **Cumplimiento:** el Constitution Check de cada plan DEBE pasar antes de implementar. Las
  violaciones se justifican en la tabla de complejidad del plan o no se aprueban. Se revisa el
  cumplimiento al cerrar cada fase del `ROADMAP.md`.

**Version**: 1.0.0 | **Ratified**: 2026-11-12 | **Last Amended**: 2026-11-12
