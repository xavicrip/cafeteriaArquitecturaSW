# Feature Specification: Plataforma de entrega continua

**Feature Branch**: `001-plataforma-entrega`
**Created**: 2026-11-12
**Status**: Draft
**Input**: User description: "Plataforma de entrega (Fase 0): flujo de Git con commits atómicos y reversibles, pipeline CI/CD con Quality Gate de SonarQube, despliegue del frontend en Vercel y del backend en Render (preview, staging, producción con aprobación manual y rollback automático), y métricas de entrega y aporte del equipo en Grafana. Basado en promtp.md §6–§9."

## User Scenarios & Testing *(mandatory)*

Actores: **desarrollador** (propone cambios), **revisor** (aprueba cambios), **responsable de release** (autoriza el paso a producción) y **líder del equipo** (sigue la entrega y el aporte del equipo).

### User Story 1 - Integrar un cambio con controles automáticos (Priority: P1)

Un desarrollador propone un cambio. Automáticamente se verifica que cada commit tenga el formato acordado y funcione por sí solo, que el código pase el análisis estático, los tests y el umbral de calidad, y que no introduzca vulnerabilidades ni secretos. El resultado aparece en la propia propuesta de cambio, y la integración queda bloqueada mientras algún control falle.

**Why this priority**: sin esta puerta, ninguno de los principios de calidad, seguridad y reversibilidad de la constitución se puede garantizar. Es la base de todo lo demás.

**Independent Test**: abrir una propuesta de cambio con un commit correcto y otra con un commit roto o mal nombrado; la primera se puede integrar y la segunda queda bloqueada con el motivo visible.

**Acceptance Scenarios**:

1. **Dado** un cambio cuyos commits cumplen el formato, compilan y pasan los tests, **Cuando** el desarrollador abre la propuesta, **Entonces** todos los controles pasan y el cambio se puede integrar tras una aprobación.
2. **Dado** un cambio con un commit intermedio que no compila, **Cuando** se ejecutan los controles, **Entonces** la integración se bloquea y se indica qué commit falla.
3. **Dado** un cambio cuya cobertura de tests en el código nuevo está por debajo del umbral, **Cuando** se evalúa la calidad, **Entonces** la integración se bloquea y se muestran las métricas que no se cumplen.
4. **Dado** un cambio que contiene un secreto o una dependencia con vulnerabilidad crítica, **Cuando** se ejecutan los controles de seguridad, **Entonces** la integración se bloquea.
5. **Dado** un mensaje de commit que no cumple el formato, **Cuando** el desarrollador intenta crear el commit en su equipo, **Entonces** recibe un aviso inmediato antes de compartirlo.

---

### User Story 2 - Ver el cambio desplegado antes y después de integrarlo (Priority: P1)

Cada propuesta de cambio obtiene un entorno de vista previa, con frontend y backend, donde se ejecutan las pruebas de punta a punta y de accesibilidad y donde el equipo puede revisar el cambio. Al integrarse, el cambio llega solo al entorno de staging, donde se ejecutan pruebas de humo.

**Why this priority**: permite validar con usuarios y pruebas reales antes de producción, y garantiza que staging siempre refleje la rama principal.

**Independent Test**: abrir una propuesta de cambio y comprobar que aparece un enlace a su vista previa; integrarla y comprobar que staging muestra la nueva versión sin intervención manual.

**Acceptance Scenarios**:

1. **Dado** una propuesta de cambio que pasa los controles de US1, **Cuando** termina la construcción, **Entonces** se publica una vista previa y su enlace aparece en la propuesta.
2. **Dado** una vista previa publicada, **Cuando** se ejecutan las pruebas de punta a punta y de accesibilidad, **Entonces** sus resultados bloquean la integración si fallan.
3. **Dado** un cambio integrado en la rama principal, **Cuando** termina el proceso, **Entonces** staging ejecuta esa versión y pasa las pruebas de humo.
4. **Dado** que el despliegue a staging falla, **Cuando** el proceso termina, **Entonces** staging conserva la versión anterior y el equipo recibe una alerta.

---

### User Story 3 - Publicar en producción con aprobación y vuelta atrás automática (Priority: P2)

El responsable de release aprueba una versión candidata y esta se publica en producción sin interrumpir el servicio: primero el backend y después el frontend. Si las pruebas de humo posteriores fallan, el sistema vuelve sola a la versión anterior.

**Why this priority**: necesario antes del lanzamiento real (Fase 6), pero no bloquea el trabajo de desarrollo de las primeras fases.

**Independent Test**: publicar una versión que pasa las pruebas de humo y otra con un fallo provocado; la primera queda en producción y la segunda se revierte sola.

**Acceptance Scenarios**:

1. **Dado** una versión candidata validada en staging, **Cuando** el responsable de release la aprueba, **Entonces** se publica en producción sin caída del servicio.
2. **Dado** que nadie ha aprobado la versión, **Cuando** está lista, **Entonces** no llega a producción.
3. **Dado** una publicación cuyas pruebas de humo fallan, **Cuando** se detecta el fallo, **Entonces** el frontend y el backend vuelven a la versión anterior y se notifica al equipo.
4. **Dado** una publicación cuya migración de datos falla, **Cuando** se intenta desplegar el backend, **Entonces** el despliegue se cancela y la versión anterior sigue atendiendo.
5. **Dado** una versión publicada, **Cuando** termina, **Entonces** existe una etiqueta de versión y un registro de cambios generado a partir de los commits.

---

### User Story 4 - Revertir cualquier cambio de forma segura (Priority: P2)

Ante un problema, el equipo revierte un commit concreto de la rama principal y el resultado se integra y despliega igual que cualquier otro cambio, sin romper la aplicación ni los datos.

**Why this priority**: es la garantía de reversibilidad del principio VII; sin ella, los commits atómicos pierden su valor.

**Independent Test**: revertir un commit reciente de la rama principal, incluido uno con una migración de datos, y comprobar que la reversión pasa todos los controles y se despliega en staging.

**Acceptance Scenarios**:

1. **Dado** cualquier commit de la rama principal, **Cuando** se revierte, **Entonces** la reversión pasa todos los controles y se despliega.
2. **Dado** un commit que añadió una migración de datos ya aplicada, **Cuando** el equipo necesita deshacerla, **Entonces** existe una guía y la reversión se hace con una nueva migración que conserva los datos.
3. **Dado** una funcionalidad incompleta integrada, **Cuando** está desactivada, **Entonces** no es visible ni accesible para los usuarios.

---

### User Story 5 - Seguir la entrega y el aporte del equipo (Priority: P3)

El líder del equipo y el propio equipo consultan paneles con las métricas de entrega (frecuencia de despliegue, tiempo desde el primer commit hasta producción, tasa de fallos de los cambios, tiempo de recuperación), el flujo de trabajo, la calidad del código, la salud del proceso de integración y el aporte de cada miembro (cambios integrados, revisiones, tests y documentación).

**Why this priority**: aporta visibilidad y mejora continua, pero el equipo puede trabajar sin ella las primeras semanas.

**Independent Test**: integrar y desplegar un cambio y verificar que aparece en los paneles de entrega y de aporte del equipo.

**Acceptance Scenarios**:

1. **Dado** un despliegue a producción, **Cuando** termina, **Entonces** el panel de entrega lo refleja y lo marca en la línea de tiempo.
2. **Dado** un miembro que hizo revisiones pero pocos commits, **Cuando** se consulta el panel de aporte, **Entonces** sus revisiones aparecen como aporte con el mismo peso visual que los commits.
3. **Dado** un usuario sin permiso, **Cuando** intenta abrir el panel de aporte individual, **Entonces** se le niega el acceso.
4. **Dado** que la rama principal falla o la tasa de aprobación del umbral de calidad baja, **Cuando** se detecta, **Entonces** se alerta al equipo.

### Edge Cases

- Una propuesta de cambio con muchos commits: la verificación commit por commit debe seguir siendo viable en tiempo (ver SC-001).
- Un test que falla de forma intermitente: se registra como inestable en el panel de salud y no se ignora en silencio.
- Dos publicaciones a producción aprobadas casi a la vez: se ejecutan en orden, nunca en paralelo.
- El servicio de análisis de calidad o el de métricas no está disponible: la integración se bloquea (calidad) o continúa y las métricas se envían después (métricas), sin perder datos.
- Una vista previa de backend no está disponible por el plan contratado: las vistas previas usan el backend de staging.
- Un cambio solo de documentación: pasa los controles aplicables sin exigir cobertura de código.

## Requirements *(mandatory)*

### Functional Requirements

**Flujo de commits**
- **FR-001**: El sistema DEBE validar que cada mensaje de commit cumple el formato acordado (tipo, alcance de módulo, descripción), en el equipo del desarrollador y de nuevo en la integración.
- **FR-002**: El sistema DEBE verificar que cada commit de una propuesta de cambio compila y pasa los tests de los módulos afectados por sí solo.
- **FR-003**: La rama principal DEBE estar protegida: sin cambios directos, con al menos una aprobación, con todos los controles en verde y con historial lineal que conserve los commits individuales.
- **FR-004**: El sistema DEBE generar la versión semántica y el registro de cambios a partir de los mensajes de commit.
- **FR-005**: El proyecto DEBE ofrecer un mecanismo para activar y desactivar funcionalidades sin desplegar, y una guía para escribir migraciones de datos compatibles con la versión anterior.

**Controles de integración**
- **FR-006**: El sistema DEBE ejecutar, en cada propuesta de cambio: análisis estático y de tipos, tests unitarios, tests de integración con servicios reales, análisis de seguridad (dependencias, secretos, imágenes) y construcción.
- **FR-007**: El sistema DEBE evaluar un umbral de calidad sobre el código nuevo con estos mínimos: cobertura ≥ 80 % (≥ 95 % en promociones, pedidos y pagos), 0 bugs y 0 vulnerabilidades nuevos, 100 % de puntos sensibles de seguridad revisados, duplicación < 3 % y calificación máxima en mantenibilidad, fiabilidad y seguridad.
- **FR-008**: El sistema DEBE bloquear la integración si falla cualquier control y mostrar el motivo en la propuesta de cambio.
- **FR-009**: El sistema DEBE ejecutar pruebas de punta a punta de los flujos críticos y pruebas automáticas de accesibilidad sobre la vista previa.

**Entornos y despliegue**
- **FR-010**: El sistema DEBE mantener tres entornos: vista previa (por propuesta de cambio), staging y producción, con datos de prueba, datos anonimizados y datos reales, respectivamente.
- **FR-011**: Solo el proceso automatizado DEBE poder desplegar; el despliegue automático por cambios en el repositorio y el despliegue manual desde equipos personales quedan desactivados.
- **FR-012**: El sistema DEBE desplegar exactamente el mismo artefacto que pasó los controles, sin reconstruirlo.
- **FR-013**: El sistema DEBE desplegar automáticamente en staging cada cambio integrado y ejecutar pruebas de humo.
- **FR-014**: La publicación en producción DEBE requerir la aprobación explícita de una persona autorizada.
- **FR-015**: El sistema DEBE aplicar las migraciones de datos antes de activar la nueva versión del backend y cancelar el despliegue si fallan.
- **FR-016**: El sistema DEBE publicar el backend antes que el frontend y sin interrumpir el servicio.
- **FR-017**: El sistema DEBE volver automáticamente a la versión anterior de frontend y backend si las pruebas de humo de producción fallan, y notificar al equipo.
- **FR-018**: El sistema DEBE ejecutar pruebas de carga programadas contra staging, al menos cada noche y antes de cada temporada de festividades.
- **FR-019**: Los secretos de cada entorno DEBEN gestionarse fuera del repositorio y ser distintos por entorno.

**Métricas del equipo**
- **FR-020**: El sistema DEBE recoger datos del repositorio (commits, propuestas de cambio, revisiones), del proceso de integración y despliegue, y del análisis de calidad.
- **FR-021**: El sistema DEBE ofrecer paneles de: métricas de entrega (las cuatro métricas DORA), flujo de trabajo, calidad, salud del proceso de integración y aporte del equipo.
- **FR-022**: El panel de aporte DEBE incluir revisiones, tests y documentación además de los commits, y NO DEBE ordenar a los miembros en un ranking.
- **FR-023**: El acceso al panel de aporte individual DEBE estar restringido a los roles autorizados, y el equipo DEBE ser informado de qué datos se recogen y para qué.
- **FR-024**: El sistema DEBE alertar al equipo cuando la rama principal falla, cuando cae la tasa de aprobación del umbral de calidad o cuando aumenta la tasa de fallos de los cambios.

### Key Entities

- **Propuesta de cambio**: conjunto de commits que se integra en la rama principal; tiene autor, revisores, estado de los controles y vista previa.
- **Commit**: cambio lógico único con tipo, alcance de módulo y descripción; debe ser reversible.
- **Ejecución del proceso**: corrida de controles o despliegue con su resultado, duración y control fallido, si lo hay.
- **Resultado de calidad**: métricas del código nuevo y veredicto del umbral.
- **Despliegue**: versión publicada en un entorno, con fecha, artefacto, aprobador (en producción) y resultado (correcto, revertido).
- **Entorno**: vista previa, staging o producción, con su configuración y sus secretos.
- **Métrica del equipo**: valor agregado por equipo o por persona y periodo.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El desarrollador conoce el resultado de todos los controles de una propuesta de cambio típica (≤ 10 commits) en menos de 15 minutos.
- **SC-002**: El 100 % de los commits de la rama principal cumple el formato acordado y pasa sus tests por sí solo.
- **SC-003**: El 100 % de los cambios integrados pasó el umbral de calidad.
- **SC-004**: Un cambio integrado está disponible en staging en menos de 20 minutos.
- **SC-005**: La vuelta atrás automática en producción deja el servicio en la versión anterior en menos de 5 minutos desde la detección del fallo.
- **SC-006**: Las publicaciones en producción no generan interrupciones del servicio visibles para los clientes.
- **SC-007**: La reversión de cualquier commit de la rama principal elegido al azar se integra y despliega en staging sin intervención adicional en el 100 % de las pruebas.
- **SC-008**: Los paneles de métricas reflejan cada despliegue en menos de 1 hora.
- **SC-009**: Al final de la Fase 0, un cambio mínimo (comprobación de salud y página de inicio) recorre todo el flujo, desde la propuesta hasta staging, y aparece en el panel de entrega.

## Assumptions

- Restricciones de la constitución (v1.0.0) que este spec no reabre: repositorio y automatización en GitHub (GitHub Actions); umbral de calidad con **SonarQube**; frontend en **Vercel**; backend en **Render**; métricas en **Grafana**; TypeScript sin Python en la aplicación.
- El equipo cuenta con planes de Vercel y Render que permiten vistas previas, migraciones previas al despliegue y autoescalado; si el plan de Render no permite vistas previas del backend, se usa staging (ver Edge Cases).
- Grafana Cloud y SonarQube Cloud se usan en su versión gestionada salvo que el equipo decida autohospedarlos (decisión pendiente en `ROADMAP.md`).
- El equipo tiene entre 2 y 4 personas; los umbrales de tiempo de SC-001 y SC-004 se calibran para ese tamaño.
- El aporte individual se usa para conversaciones de mejora y reconocimiento, nunca para evaluaciones automáticas.
- Las pruebas de carga se ejecutan contra staging; nunca contra producción con clientes reales.
