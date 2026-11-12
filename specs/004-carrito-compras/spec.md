# Feature Specification: Carrito de compras

**Feature Branch**: `004-carrito-compras`
**Created**: 2026-11-12
**Status**: Draft
**Input**: User description: "RF-03 Carrito de compras: un carrito activo por cliente, persistente entre sesiones y dispositivos; agregar, quitar y cambiar cantidades de varios productos; muestra el precio con descuentos aplicados y avisa si un producto dejó de estar disponible o cambió de precio."

## User Scenarios & Testing *(mandatory)*

Actor: **cliente** con sesión iniciada.

### User Story 1 - Armar el carrito (Priority: P1)

El cliente añade varios productos del menú a su carrito, cambia cantidades y quita productos, viendo en todo momento el subtotal, los descuentos y el total.

**Why this priority**: es el paso previo imprescindible al pedido.

**Independent Test**: añadir tres productos distintos, cambiar una cantidad, quitar uno y comprobar que el total se recalcula correctamente.

**Acceptance Scenarios**:

1. **Dado** un producto disponible, **Cuando** el cliente lo añade, **Entonces** aparece en el carrito y se anuncia el cambio a los lectores de pantalla.
2. **Dado** un producto ya en el carrito, **Cuando** se añade otra vez, **Entonces** aumenta su cantidad en lugar de duplicarse.
3. **Dado** un carrito con productos, **Cuando** el cliente cambia una cantidad o quita un producto, **Entonces** el total se recalcula de inmediato.
4. **Dado** un producto con una promoción vigente, **Cuando** está en el carrito, **Entonces** se muestran el precio original, el descuento y el precio final.
5. **Dado** un cliente, **Cuando** quita un producto por error, **Entonces** puede deshacer la acción.

---

### User Story 2 - Recuperar el carrito en otro momento o dispositivo (Priority: P1)

El cliente arma su carrito en el móvil y lo encuentra igual al abrir la tienda en el ordenador, o al volver días después.

**Why this priority**: es un requisito explícito (carrito por cliente) y evita compras abandonadas.

**Independent Test**: añadir productos en un dispositivo, iniciar sesión en otro y ver el mismo carrito.

**Acceptance Scenarios**:

1. **Dado** un carrito con productos, **Cuando** el cliente inicia sesión en otro dispositivo, **Entonces** ve el mismo carrito.
2. **Dado** que el cliente modifica el carrito en dos dispositivos, **Cuando** abre cualquiera de ellos, **Entonces** ve el estado más reciente.
3. **Dado** un carrito sin actividad durante 30 días, **Cuando** vence ese plazo, **Entonces** se vacía.

---

### User Story 3 - Enterarse de cambios antes de pagar (Priority: P2)

Si un producto del carrito se agota, se retira o cambia de precio, o una promoción termina, el carrito lo avisa claramente y el cliente decide qué hacer antes de confirmar.

**Why this priority**: evita sorpresas en el pago y reclamos; los menús y promociones cambian con frecuencia.

**Independent Test**: con un producto en el carrito, cambiar su precio desde la administración y comprobar el aviso en el carrito.

**Acceptance Scenarios**:

1. **Dado** un producto del carrito que se agota, **Cuando** el cliente abre el carrito, **Entonces** se le avisa y no puede confirmar el pedido hasta quitarlo.
2. **Dado** un producto cuyo precio cambió, **Cuando** el cliente abre el carrito, **Entonces** ve el precio anterior y el nuevo, y el total actualizado.
3. **Dado** una promoción que terminó, **Cuando** el cliente abre el carrito, **Entonces** se le avisa de que el descuento ya no aplica.
4. **Dado** cualquiera de estos avisos, **Cuando** aparece, **Entonces** se anuncia a los lectores de pantalla sin mover el foco del cliente.

### Edge Cases

- Cantidad mayor que las existencias disponibles: se limita a las existencias y se explica por qué.
- Cantidad cero o negativa: se trata como quitar el producto (cero) o se rechaza (negativa).
- Carrito con un producto de un menú fuera de horario: se avisa y no se puede confirmar hasta que el menú esté disponible o se quite el producto.
- Límite de cantidad por producto y por pedido (p. ej. 20 unidades por producto) para frenar abusos.
- La sesión caduca mientras el cliente edita el carrito: el carrito no se pierde.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE mantener un único carrito activo por cliente.
- **FR-002**: El cliente DEBE poder añadir productos disponibles, cambiar cantidades, quitar productos, deshacer la última eliminación y vaciar el carrito.
- **FR-003**: El carrito DEBE conservarse entre sesiones y dispositivos del mismo cliente, y vaciarse tras 30 días sin actividad.
- **FR-004**: El carrito DEBE mostrar por producto el precio base, el descuento aplicado y el precio final, y en total el subtotal, el descuento total y el total a pagar.
- **FR-005**: Todos los precios y descuentos del carrito DEBEN calcularse en el servidor con las reglas vigentes; el cliente nunca puede fijar un precio.
- **FR-006**: El sistema DEBE detectar productos agotados, retirados, fuera de horario o con precio cambiado y promociones finalizadas, y avisar al cliente.
- **FR-007**: El sistema NO DEBE permitir confirmar un pedido con productos no disponibles.
- **FR-008**: El sistema DEBE limitar la cantidad por producto a las existencias disponibles y a un máximo configurable.
- **FR-009**: Los cambios en el carrito y los avisos DEBEN anunciarse a los lectores de pantalla sin mover el foco, y cumplir WCAG 2.2 AAA.

### Key Entities

- **Carrito**: pertenece a un cliente; tiene líneas, fecha de última actividad y totales calculados.
- **Línea de carrito**: producto, cantidad, precio base, descuento aplicado y precio final en el momento del cálculo.
- **Aviso de carrito**: tipo (agotado, retirado, fuera de horario, cambio de precio, fin de promoción) y producto afectado.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El cliente ve el total actualizado en menos de 1 segundo tras cada cambio en el carrito.
- **SC-002**: El carrito es idéntico en todos los dispositivos del cliente en menos de 5 segundos tras un cambio.
- **SC-003**: El 100 % de los totales mostrados coincide con el total cobrado al confirmar, salvo que haya un aviso de cambio visible.
- **SC-004**: 0 pedidos confirmados contienen productos no disponibles.
- **SC-005**: El 100 % de las operaciones del carrito se puede completar solo con teclado y con lector de pantalla.

## Assumptions

- Solo los clientes con sesión iniciada tienen carrito; no hay carrito de invitado en la primera versión.
- Las existencias no se reservan al añadir al carrito, sino al confirmar el pedido (spec 006).
- El cálculo de descuentos lo define el spec 005; este spec solo muestra su resultado.
- Los límites por defecto son 20 unidades por producto y 50 por carrito, configurables por el administrador.
