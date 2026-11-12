# Feature Specification: Promociones y descuentos

**Feature Branch**: `005-promociones-descuentos`
**Created**: 2026-11-12
**Status**: Draft
**Input**: User description: "RF-06 Promociones y descuentos: el administrador define promociones (porcentaje, monto fijo, 2x1, combo) con alcance por producto, categoría o menú, ventana de vigencia (p. ej. Navidad, Día de la Madre), condiciones opcionales (monto mínimo, cupón) y reglas de combinación (prioridad, acumulable o no). El precio final siempre lo calcula el servidor."

## User Scenarios & Testing *(mandatory)*

Actores: **administrador** y **cliente**.

### User Story 1 - Programar una promoción para una festividad (Priority: P1)

El administrador crea una promoción (p. ej. "20 % en bebidas calientes del 20 al 25 de diciembre") con su tipo, alcance y fechas. La promoción se activa y desactiva sola en su ventana de vigencia.

**Why this priority**: las promociones por festividades son el requisito central de esta funcionalidad y la principal causa de picos de demanda.

**Independent Test**: programar una promoción para una ventana futura corta y comprobar que se aplica solo dentro de ella.

**Acceptance Scenarios**:

1. **Dado** una promoción programada, **Cuando** llega su fecha y hora de inicio, **Entonces** se aplica a los productos de su alcance sin intervención manual.
2. **Dado** una promoción vigente, **Cuando** llega su fecha y hora de fin, **Entonces** deja de aplicarse.
3. **Dado** una promoción fuera de su ventana, **Cuando** un cliente ve el menú, **Entonces** no se le aplica ni se muestra como vigente.
4. **Dado** un administrador, **Cuando** crea una promoción con fecha de fin anterior a la de inicio o sin alcance, **Entonces** se le indica el error y no se guarda.
5. **Dado** una promoción vigente, **Cuando** un cliente ve un producto afectado, **Entonces** ve el precio original, el precio con descuento y el nombre de la promoción.

---

### User Story 2 - Usar distintos tipos de descuento (Priority: P1)

El administrador elige entre porcentaje, monto fijo, 2x1 (o N×M) y combo (precio especial por un conjunto de productos), aplicados a un producto, una categoría o un menú completo.

**Why this priority**: son los tipos de promoción que pide el negocio.

**Independent Test**: crear una promoción de cada tipo y verificar el precio final en carritos de prueba.

**Acceptance Scenarios**:

1. **Dado** un 20 % sobre una categoría, **Cuando** un cliente añade un producto de esa categoría, **Entonces** se aplica el 20 % a ese producto.
2. **Dado** un descuento de monto fijo mayor que el precio del producto, **Cuando** se aplica, **Entonces** el precio final es 0 y nunca negativo.
3. **Dado** un 2x1 en un producto, **Cuando** el cliente añade 3 unidades, **Entonces** paga 2.
4. **Dado** un combo "café + sánduche a precio especial", **Cuando** el carrito contiene ambos productos, **Entonces** se aplica el precio del combo a esa pareja.

---

### User Story 3 - Combinar promociones de forma predecible (Priority: P2)

Cuando varias promociones afectan al mismo producto o pedido, el sistema aplica reglas claras de prioridad y acumulación, y el resultado es siempre el mismo para el mismo carrito.

**Why this priority**: sin reglas explícitas, los precios son impredecibles y pueden causar pérdidas.

**Independent Test**: crear dos promociones solapadas (una acumulable y otra no) y verificar el precio resultante en varios carritos.

**Acceptance Scenarios**:

1. **Dado** dos promociones no acumulables sobre el mismo producto, **Cuando** ambas están vigentes, **Entonces** se aplica la de mayor prioridad y, a igual prioridad, la más favorable para el cliente.
2. **Dado** una promoción acumulable y otra no acumulable, **Cuando** coinciden, **Entonces** se siguen las reglas de combinación definidas y el carrito muestra qué promociones se aplicaron.
3. **Dado** el mismo carrito y el mismo momento, **Cuando** se calcula el precio varias veces, **Entonces** el resultado es siempre idéntico.

---

### User Story 4 - Cupones y condiciones (Priority: P3)

El administrador crea promociones que solo se aplican con un código de cupón o a partir de un monto mínimo, con límite de usos opcional.

**Why this priority**: útil para campañas, pero no imprescindible para las festividades.

**Independent Test**: crear un cupón con monto mínimo y límite de usos, y probar su aplicación por debajo y por encima del mínimo y tras agotar los usos.

**Acceptance Scenarios**:

1. **Dado** un cupón válido, **Cuando** el cliente lo introduce y cumple las condiciones, **Entonces** se aplica el descuento.
2. **Dado** un carrito por debajo del monto mínimo, **Cuando** el cliente introduce el cupón, **Entonces** se le indica cuánto le falta para alcanzarlo.
3. **Dado** un cupón con límite de usos agotado, **Cuando** alguien lo introduce, **Entonces** se le informa de que ya no está disponible.
4. **Dado** varios intentos con códigos inválidos, **Cuando** se supera un límite, **Entonces** se frenan los intentos para evitar que se adivinen códigos.

### Edge Cases

- Una promoción empieza o termina mientras el cliente tiene el carrito abierto: el carrito se recalcula y avisa del cambio (spec 004); el precio que cuenta es el del momento de confirmar el pedido.
- Redondeo: los descuentos se redondean al centavo con una regla única y documentada.
- Un producto pertenece a varias categorías con promociones distintas: se aplican las reglas de combinación.
- El administrador edita una promoción ya vigente: el cambio aplica a partir de ese momento; los pedidos ya confirmados no cambian.
- Cambio de horario o zona horaria: las ventanas se interpretan en la hora local de la cafetería.
- El combo se cumple varias veces en el mismo carrito: se aplica tantas veces como se complete.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Los administradores DEBEN poder crear, editar, pausar y archivar promociones.
- **FR-002**: El sistema DEBE soportar los tipos porcentaje, monto fijo, N×M (p. ej. 2x1, 3x2) y combo.
- **FR-003**: El alcance de una promoción DEBE poder ser uno o varios productos, una o varias categorías o un menú completo.
- **FR-004**: Cada promoción DEBE tener una ventana de vigencia (fecha y hora de inicio y fin) y, opcionalmente, una etiqueta de festividad para agruparla en reportes.
- **FR-005**: El sistema DEBE activar y desactivar las promociones automáticamente según su ventana, con una precisión de 1 minuto.
- **FR-006**: El sistema DEBE soportar condiciones opcionales: monto mínimo del pedido, código de cupón y límite total de usos.
- **FR-007**: Cada promoción DEBE indicar su prioridad y si es acumulable; el sistema DEBE aplicar estas reglas de forma determinista.
- **FR-008**: El precio final NUNCA DEBE ser negativo.
- **FR-009**: El cálculo de precios DEBE hacerse siempre en el servidor, con aritmética exacta y una regla de redondeo única.
- **FR-010**: El sistema DEBE mostrar al cliente qué promociones se aplicaron y cuánto ahorró.
- **FR-011**: Al confirmar un pedido, el sistema DEBE guardar las promociones aplicadas y sus importes de forma inmutable (spec 006).
- **FR-012**: El sistema DEBE limitar los intentos de introducción de cupones para evitar ataques de adivinación.
- **FR-013**: Los administradores DEBEN poder simular el precio de un carrito de ejemplo en una fecha y hora futuras antes de activar una promoción.
- **FR-014**: Todos los cambios en promociones DEBEN quedar en el registro de auditoría.

### Key Entities

- **Promoción**: nombre, tipo, valor, alcance, ventana de vigencia, condiciones, prioridad, acumulabilidad, estado y etiqueta de festividad.
- **Cupón**: código asociado a una promoción, con límite y número de usos.
- **Descuento aplicado**: promoción, línea o pedido afectado e importe descontado; se guarda en el pedido.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 100 % de las promociones programadas se activan y desactivan dentro del minuto previsto.
- **SC-002**: El 100 % de los carritos de la batería de pruebas de combinación produce el precio esperado.
- **SC-003**: Un administrador programa una promoción de festividad en menos de 5 minutos.
- **SC-004**: 0 pedidos con un precio final negativo o calculado con una promoción no vigente.
- **SC-005**: El cálculo del precio no añade una espera perceptible al cliente (menos de 300 ms al actualizar el carrito), también durante los picos de festividades.

## Assumptions

- Las promociones se aplican automáticamente sin necesidad de que el cliente haga nada, salvo las que requieren cupón.
- Por defecto, las promociones no son acumulables, salvo que el administrador indique lo contrario.
- Los cupones son de uso compartido (un mismo código para todos); los cupones personales de un solo uso quedan fuera de la primera versión.
- Programas de fidelidad o puntos quedan fuera del alcance.
- Los montos se expresan en dólares estadounidenses (moneda de Ecuador) con dos decimales.
