# Feature Specification: Pedidos

**Feature Branch**: `006-pedidos`
**Created**: 2026-11-12
**Status**: Draft
**Input**: User description: "RF-04 Pedidos: al confirmar el carrito se crea un pedido con estados pendiente_pago → pagado → en_preparacion → listo → entregado, más cancelado. El cliente ve el estado en tiempo (casi) real; el personal ve una cola de pedidos y cambia su estado. Debe soportar alto volumen de pedidos simultáneos sin perder ni duplicar pedidos."

## User Scenarios & Testing *(mandatory)*

Actores: **cliente** y **personal de cafetería**.

### User Story 1 - Confirmar un pedido (Priority: P1)

El cliente revisa su carrito, elige cómo recibirlo y confirma. El sistema crea un pedido con los precios vigentes en ese momento y lo lleva al paso de pago.

**Why this priority**: es el núcleo del negocio.

**Independent Test**: con un carrito válido, confirmar el pedido y verificar que se crea un único pedido con los precios correctos, aunque se pulse confirmar varias veces o se pierda la conexión.

**Acceptance Scenarios**:

1. **Dado** un carrito con productos disponibles, **Cuando** el cliente confirma, **Entonces** se crea un pedido en estado *pendiente de pago* con número de pedido, y el carrito se vacía.
2. **Dado** que el cliente pulsa confirmar varias veces o reintenta tras perder la conexión, **Cuando** el sistema recibe las peticiones, **Entonces** se crea un solo pedido.
3. **Dado** que un producto se agotó justo antes de confirmar, **Cuando** el cliente confirma, **Entonces** el pedido no se crea, se le indica qué producto falta y su carrito se conserva.
4. **Dado** un pedido creado, **Cuando** se consulta más tarde, **Entonces** conserva los precios, descuentos y descripciones del momento de la compra.
5. **Dado** un pedido confirmado, **Cuando** el cliente ve la confirmación, **Entonces** se le indica que el pedido se retira en la cafetería y recibe un código de retiro con el número de pedido.

---

### User Story 2 - Seguir el estado del pedido (Priority: P1)

El cliente ve en todo momento en qué estado está su pedido y recibe un aviso cuando está listo.

**Why this priority**: reduce la incertidumbre del cliente y las consultas al personal.

**Independent Test**: cambiar el estado de un pedido desde la vista del personal y comprobar que el cliente lo ve en segundos sin recargar.

**Acceptance Scenarios**:

1. **Dado** un pedido en curso, **Cuando** cambia de estado, **Entonces** el cliente lo ve en menos de 5 segundos sin recargar la página.
2. **Dado** un pedido que pasa a *listo*, **Cuando** cambia, **Entonces** el cliente recibe una notificación (en la aplicación y por correo).
3. **Dado** un cambio de estado, **Cuando** se muestra, **Entonces** se anuncia a los lectores de pantalla y se indica con texto, no solo con color.
4. **Dado** que se pierde la conexión en tiempo real, **Cuando** se recupera, **Entonces** el cliente ve el estado actual sin perder cambios.

---

### User Story 3 - Gestionar la cola de pedidos (Priority: P1)

El personal ve los pedidos pagados en orden de llegada, los pasa a *en preparación*, *listo* y *entregado*, y ve en el momento los nuevos pedidos.

**Why this priority**: sin esta cola, los pedidos no se preparan.

**Independent Test**: varios clientes hacen pedidos; el personal los ve aparecer en orden y los avanza hasta *entregado*.

**Acceptance Scenarios**:

1. **Dado** un pedido pagado, **Cuando** se confirma su pago, **Entonces** aparece en la cola del personal en segundos.
2. **Dado** un pedido en la cola, **Cuando** el personal lo avanza de estado, **Entonces** el cambio queda registrado con quién y cuándo.
3. **Dado** un pedido, **Cuando** el personal intenta un cambio de estado no permitido (p. ej. de *pendiente de pago* a *listo*), **Entonces** se le impide.
4. **Dado** dos miembros del personal que cambian el mismo pedido a la vez, **Cuando** ocurre, **Entonces** solo se aplica un cambio y el otro recibe el estado actualizado.

---

### User Story 4 - Cancelar un pedido (Priority: P2)

El cliente puede cancelar un pedido mientras no esté en preparación; el personal puede cancelarlo indicando un motivo. Si estaba pagado, se inicia el reembolso.

**Why this priority**: necesario para operar, pero menos frecuente que el flujo principal.

**Independent Test**: cancelar un pedido pagado antes de su preparación y verificar que pasa a *cancelado* y se inicia el reembolso.

**Acceptance Scenarios**:

1. **Dado** un pedido pendiente de pago o pagado, no en preparación, **Cuando** el cliente lo cancela, **Entonces** pasa a *cancelado* y, si estaba pagado, se inicia el reembolso (spec 007).
2. **Dado** un pedido en preparación, **Cuando** el cliente intenta cancelarlo, **Entonces** se le informa de que ya no es posible y cómo contactar con la cafetería.
3. **Dado** un pedido pendiente de pago, **Cuando** pasan 30 minutos sin pago, **Entonces** se cancela automáticamente y se liberan las existencias reservadas.

### Edge Cases

- Pico de pedidos en una festividad: ningún pedido se pierde; si el sistema está saturado, el cliente ve un tiempo de espera estimado en lugar de un error.
- Fallo del sistema justo después de crear el pedido: al recuperarse, el pedido existe una sola vez y el resto del flujo (cola, notificaciones) continúa.
- Una notificación se envía dos veces por un reintento interno: el cliente no recibe avisos duplicados.
- El pago se confirma después de la cancelación automática por tiempo: el pedido se reactiva si hay existencias o se reembolsa automáticamente.
- El cliente elimina su cuenta con un pedido en curso: el pedido termina normalmente y después se anonimiza (spec 002).

## Clarifications

### Session 2026-11-12

- Q: ¿Cómo recibe el cliente su pedido: retiro en la cafetería, entrega a domicilio o ambos? → A: Solo retiro en la cafetería (opción A). La entrega a domicilio queda fuera de la primera versión.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Al confirmar el carrito, el sistema DEBE crear un pedido con número único, recalcular precios y descuentos vigentes y guardar una copia inmutable de productos, precios, descuentos y totales.
- **FR-002**: La confirmación DEBE ser idempotente: reintentos de la misma confirmación producen un único pedido.
- **FR-003**: El sistema DEBE verificar la disponibilidad y reservar las existencias al crear el pedido; si falta algún producto, el pedido no se crea y el carrito se conserva.
- **FR-004**: El pedido DEBE seguir la máquina de estados: *pendiente de pago* → *pagado* → *en preparación* → *listo* → *entregado*, con *cancelado* posible desde *pendiente de pago* y *pagado*, y por el personal con motivo desde *en preparación*.
- **FR-005**: El sistema DEBE rechazar cualquier transición de estado no permitida.
- **FR-006**: Cada cambio de estado DEBE registrarse con fecha, autor y motivo (si aplica), y notificarse a los módulos interesados sin perder ni duplicar eventos.
- **FR-007**: El cliente DEBE ver el estado de sus pedidos actualizado en menos de 5 segundos y recibir una notificación cuando el pedido esté listo.
- **FR-008**: El personal DEBE ver una cola de pedidos pagados en orden de llegada, actualizada en tiempo real, y poder avanzar su estado.
- **FR-009**: El sistema DEBE resolver los cambios concurrentes sobre un mismo pedido sin estados inconsistentes.
- **FR-010**: Los pedidos pendientes de pago DEBEN cancelarse automáticamente tras 30 minutos (configurable) y liberar las existencias.
- **FR-011**: El cliente DEBE poder cancelar sus pedidos que no estén en preparación; si estaban pagados, el sistema inicia el reembolso.
- **FR-012**: El sistema DEBE aceptar ≥ 1 000 pedidos por minuto en pico sin perder ni duplicar pedidos.
- **FR-013**: Los pedidos DEBEN retirarse en la cafetería: el sistema genera un código de retiro por pedido, y el personal lo verifica antes de marcar el pedido como *entregado*.
- **FR-014**: Las pantallas del cliente y la cola del personal DEBEN cumplir WCAG 2.2 AAA.

### Key Entities

- **Pedido**: número, código de retiro, cliente, estado, fecha, totales y forma de pago.
- **Línea de pedido**: copia inmutable de producto, cantidad, precio, descuentos y versión del menú.
- **Cambio de estado**: estado anterior, estado nuevo, autor, fecha y motivo.
- **Reserva de existencias**: productos y cantidades reservadas por un pedido hasta su pago o cancelación.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El cliente confirma un pedido en menos de 1 segundo desde que pulsa confirmar (sin contar el pago), en el 95 % de los casos, también en picos.
- **SC-002**: En la prueba de carga de 1 000 pedidos por minuto, 0 pedidos perdidos y 0 duplicados.
- **SC-003**: En pruebas de fallo (reinicio de componentes, reintentos), 0 pedidos perdidos o duplicados y 0 notificaciones duplicadas.
- **SC-004**: El cliente y el personal ven los cambios de estado en menos de 5 segundos.
- **SC-005**: El 100 % de los pedidos conserva los precios del momento de la compra.
- **SC-006**: El personal avanza un pedido de estado con una sola acción.

## Assumptions

- Hay una sola cafetería; todos los pedidos van a la misma cola.
- Los pedidos solo se retiran en la cafetería; la entrega a domicilio queda fuera de la primera versión (aclaración del 2026-11-12). El estado *entregado* significa que el cliente retiró el pedido.
- Los pedidos se hacen para el momento (lo antes posible); los pedidos programados para más tarde quedan fuera de la primera versión.
- El tiempo máximo de espera de pago es de 30 minutos y es configurable.
- El reembolso lo ejecuta el módulo de pagos (spec 007); este spec solo lo inicia.
