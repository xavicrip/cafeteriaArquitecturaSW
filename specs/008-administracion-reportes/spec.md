# Feature Specification: Administración, reportes y auditoría

**Feature Branch**: `008-administracion-reportes`
**Created**: 2026-11-12
**Status**: Draft
**Input**: User description: "RF-07 Administración y reportes: panel de administración para menús, promociones, pedidos y usuarios del personal; reportes de ventas por periodo, productos más vendidos y uso de promociones; registro de auditoría de cambios en precios, menús, promociones y pagos."

## User Scenarios & Testing *(mandatory)*

Actores: **administrador** y **personal de cafetería**.

### User Story 1 - Ver cómo va el negocio (Priority: P1)

El administrador consulta las ventas de un periodo (día, semana, mes o rango), los productos más vendidos y la venta por forma de pago, y puede exportarlas.

**Why this priority**: es la información que el negocio necesita a diario para decidir.

**Independent Test**: con pedidos de prueba de varios días, consultar las ventas de una semana y comprobar que los totales coinciden con los pedidos.

**Acceptance Scenarios**:

1. **Dado** pedidos entregados en un periodo, **Cuando** el administrador consulta las ventas, **Entonces** ve el total, el número de pedidos, el ticket medio y el desglose por día y por forma de pago.
2. **Dado** un periodo, **Cuando** consulta los productos más vendidos, **Entonces** ve el ranking por unidades y por importe.
3. **Dado** un reporte, **Cuando** lo exporta, **Entonces** obtiene un archivo de hoja de cálculo con los mismos datos.
4. **Dado** un reporte con gráficos, **Cuando** se usa un lector de pantalla, **Entonces** los mismos datos están disponibles en una tabla accesible.

---

### User Story 2 - Medir las promociones (Priority: P2)

El administrador ve cuántas veces se usó cada promoción, cuánto se descontó y cuánto se vendió con ella, agrupado por festividad.

**Why this priority**: permite decidir qué promociones repetir en la siguiente temporada.

**Independent Test**: con pedidos con y sin promociones, consultar el reporte de una festividad y verificar usos, descuento total y ventas.

**Acceptance Scenarios**:

1. **Dado** una festividad terminada, **Cuando** el administrador consulta su reporte, **Entonces** ve por promoción: usos, descuento total, ventas asociadas y comparación con el mismo periodo sin promoción.
2. **Dado** una promoción con cupón, **Cuando** se consulta, **Entonces** se ven los usos frente al límite.

---

### User Story 3 - Saber quién cambió qué (Priority: P2)

El administrador consulta el registro de auditoría de cambios en precios, menús, promociones, pagos, roles y cuentas del personal, y lo filtra por fecha, persona o tipo de cambio.

**Why this priority**: necesario para la seguridad, la resolución de reclamos y el control interno.

**Independent Test**: cambiar un precio y confirmar una transferencia, y encontrar ambos eventos en el registro con autor, fecha y valores anterior y nuevo.

**Acceptance Scenarios**:

1. **Dado** un cambio de precio, **Cuando** el administrador consulta la auditoría, **Entonces** ve quién lo hizo, cuándo, el valor anterior y el nuevo.
2. **Dado** el registro de auditoría, **Cuando** alguien intenta modificar o borrar una entrada, **Entonces** no es posible.
3. **Dado** un miembro del personal, **Cuando** intenta ver la auditoría, **Entonces** se le deniega.

---

### User Story 4 - Operar desde un único panel (Priority: P3)

El administrador accede desde un único panel a los menús (spec 003), las promociones (spec 005), los pedidos (spec 006), los pagos pendientes (spec 007), las cuentas del personal (spec 002) y la configuración (formas de pago activas, límites, plazos).

**Why this priority**: mejora la eficiencia, pero cada función ya es usable desde su propia sección.

**Independent Test**: un administrador realiza una jornada típica (publicar un cambio de menú, revisar pedidos, confirmar una transferencia, crear una cuenta de personal) solo desde el panel.

**Acceptance Scenarios**:

1. **Dado** un administrador, **Cuando** entra al panel, **Entonces** ve un resumen del día (ventas, pedidos en curso, pagos pendientes de verificar) y accede a cada sección.
2. **Dado** un administrador, **Cuando** cambia un parámetro de configuración, **Entonces** se aplica sin desplegar y queda en la auditoría.

### Edge Cases

- Un periodo sin ventas: el reporte lo indica claramente en lugar de mostrar gráficos vacíos.
- Pedidos de clientes que eliminaron su cuenta: cuentan en los reportes, pero de forma anónima.
- Pedidos cancelados y reembolsados: se excluyen de las ventas y se muestran aparte.
- Reportes de periodos muy largos: se generan sin afectar a la velocidad de la tienda.
- Zona horaria: los periodos se calculan en la hora local de la cafetería.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE ofrecer un panel de administración con acceso a menús, promociones, pedidos, pagos pendientes, cuentas del personal, configuración, reportes y auditoría.
- **FR-002**: El sistema DEBE generar reportes de ventas por periodo con total, número de pedidos, ticket medio y desglose por día y por forma de pago, excluyendo pedidos cancelados o reembolsados.
- **FR-003**: El sistema DEBE generar el ranking de productos más vendidos por unidades y por importe.
- **FR-004**: El sistema DEBE generar reportes de uso de promociones (usos, descuento total, ventas asociadas) agrupables por festividad.
- **FR-005**: Los reportes DEBEN poder exportarse en formato de hoja de cálculo.
- **FR-006**: Los reportes NO DEBEN mostrar datos personales de clientes; solo agregados o identificadores anónimos.
- **FR-007**: La generación de reportes NO DEBE degradar el rendimiento de la tienda.
- **FR-008**: El sistema DEBE mantener un registro de auditoría inmutable de cambios en precios, menús, promociones, pagos, configuración, roles y cuentas del personal, con autor, fecha, valor anterior y valor nuevo.
- **FR-009**: Solo los administradores DEBEN poder ver los reportes y la auditoría.
- **FR-010**: Los administradores DEBEN poder modificar la configuración operativa (formas de pago activas, límites, plazos) sin desplegar.
- **FR-011**: Todo gráfico DEBE tener una alternativa en tabla accesible, y el panel DEBE cumplir WCAG 2.2 AAA.

### Key Entities

- **Reporte**: tipo, periodo, filtros y resultados agregados.
- **Entrada de auditoría**: autor, fecha, acción, entidad afectada, valor anterior y valor nuevo; inmutable.
- **Parámetro de configuración**: nombre, valor, autor y fecha del último cambio.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Los reportes de hasta un año de datos se muestran en menos de 5 segundos.
- **SC-002**: Los totales de los reportes coinciden al centavo con la suma de los pedidos del periodo.
- **SC-003**: El 100 % de los cambios en precios, promociones, pagos y roles aparece en la auditoría.
- **SC-004**: Un administrador encuentra quién hizo un cambio concreto en menos de 1 minuto.
- **SC-005**: Generar reportes durante un pico de pedidos no aumenta el tiempo de respuesta de la tienda para los clientes.

## Assumptions

- Los reportes pueden reflejar los datos con un retraso de hasta 5 minutos respecto al momento actual.
- La auditoría se conserva al menos 5 años, salvo los datos personales que contenga, que siguen la política de retención LOPDP.
- Los reportes contables o fiscales oficiales quedan fuera del alcance; el sistema exporta los datos para el sistema contable.
- Hay un solo nivel de administrador; roles de administración más finos quedan fuera de la primera versión.
