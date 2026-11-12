# Feature Specification: Menú dinámico

**Feature Branch**: `003-menu-dinamico`
**Created**: 2026-11-12
**Status**: Draft
**Input**: User description: "RF-02 Menú dinámico: el administrador crea, edita, publica y retira menús, categorías y productos (nombre, descripción, precio, imagen con texto alternativo, alérgenos, disponibilidad). Los cambios se reflejan sin desplegar código, los menús tienen versiones, y hay disponibilidad por horario y por existencias."

## User Scenarios & Testing *(mandatory)*

Actores: **administrador**, **personal de cafetería** y **cliente**.

### User Story 1 - Consultar el menú (Priority: P1)

Un cliente, con o sin sesión, abre la tienda y ve los menús vigentes en ese momento, organizados por categorías, con nombre, descripción, precio, imagen, alérgenos y disponibilidad de cada producto.

**Why this priority**: es la puerta de entrada a la compra y la página con más tráfico.

**Independent Test**: cargar un menú de prueba y verificar que un cliente lo navega, filtra por categoría y ve los alérgenos, solo con teclado y con lector de pantalla.

**Acceptance Scenarios**:

1. **Dado** un menú publicado y vigente, **Cuando** un cliente abre la tienda, **Entonces** ve sus categorías y productos con precio, imagen con texto alternativo y alérgenos.
2. **Dado** un producto agotado, **Cuando** se muestra, **Entonces** aparece como no disponible, con texto (no solo con color), y no se puede añadir al carrito.
3. **Dado** un menú con horario (p. ej. desayuno de 07:00 a 11:00), **Cuando** un cliente consulta fuera de ese horario, **Entonces** no lo ve como disponible y se le indica cuándo lo estará.
4. **Dado** un cliente con alergias, **Cuando** filtra por alérgenos a excluir, **Entonces** solo ve productos que no los contienen.

---

### User Story 2 - Crear y publicar un menú (Priority: P1)

El administrador crea un menú con categorías y productos, lo revisa en un borrador y lo publica. Los clientes lo ven en segundos, sin intervención técnica.

**Why this priority**: los menús dinámicos son un requisito central del producto.

**Independent Test**: crear un menú en borrador, comprobar que los clientes no lo ven, publicarlo y comprobar que aparece en la tienda.

**Acceptance Scenarios**:

1. **Dado** un administrador, **Cuando** crea un menú, **Entonces** queda en borrador y no es visible para los clientes.
2. **Dado** un producto sin texto alternativo en su imagen o sin información de alérgenos, **Cuando** se intenta publicar, **Entonces** la publicación se bloquea indicando qué falta.
3. **Dado** un borrador completo, **Cuando** el administrador lo publica, **Entonces** los clientes lo ven en menos de 10 segundos.
4. **Dado** un administrador, **Cuando** previsualiza un borrador, **Entonces** lo ve tal como lo verá el cliente.

---

### User Story 3 - Modificar un menú publicado sin afectar pedidos (Priority: P1)

El administrador cambia precios, descripciones o productos de un menú publicado. Se crea una nueva versión; los pedidos ya realizados conservan los datos con los que se compraron.

**Why this priority**: sin versiones, un cambio de precio alteraría pedidos ya pagados.

**Independent Test**: hacer un pedido, cambiar el precio del producto y verificar que el pedido mantiene el precio original y el menú muestra el nuevo.

**Acceptance Scenarios**:

1. **Dado** un menú publicado, **Cuando** el administrador cambia un precio y publica, **Entonces** se crea una nueva versión y la tienda muestra el nuevo precio.
2. **Dado** un pedido hecho con la versión anterior, **Cuando** se consulta, **Entonces** muestra el precio y la descripción originales.
3. **Dado** una versión publicada con un error, **Cuando** el administrador restaura la versión anterior, **Entonces** la tienda vuelve a mostrarla.
4. **Dado** un menú, **Cuando** se consulta su historial, **Entonces** se ve quién publicó cada versión, cuándo y qué cambió.

---

### User Story 4 - Gestionar la disponibilidad del día (Priority: P2)

El personal marca un producto como agotado o disponible durante la jornada, o ajusta sus existencias, sin necesidad de un administrador.

**Why this priority**: evita vender lo que no hay, pero se puede operar al principio solo con agotado/disponible.

**Independent Test**: el personal marca un producto como agotado y deja de poder añadirse al carrito en la tienda.

**Acceptance Scenarios**:

1. **Dado** un miembro del personal, **Cuando** marca un producto como agotado, **Entonces** deja de poder añadirse al carrito en segundos.
2. **Dado** un producto con existencias, **Cuando** llegan a cero por las ventas, **Entonces** pasa a agotado automáticamente.
3. **Dado** un miembro del personal, **Cuando** intenta cambiar un precio, **Entonces** se le deniega (solo administradores).

### Edge Cases

- Un cliente tiene en el carrito un producto que se retira o se agota: el carrito lo avisa (ver spec 004) y no permite confirmar ese producto.
- Dos administradores editan el mismo menú a la vez: el segundo en guardar es avisado del conflicto y no sobrescribe en silencio.
- Dos menús vigentes se solapan en horario: se muestran ambos, y un producto presente en los dos aparece una sola vez por categoría.
- Zona horaria: los horarios se interpretan siempre en la hora local de la cafetería.
- Un menú sin productos disponibles: se muestra con un mensaje claro, no como una página vacía.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Los administradores DEBEN poder crear, editar, publicar, retirar y restaurar menús, categorías y productos.
- **FR-002**: Cada producto DEBE tener nombre, descripción, precio, imagen con texto alternativo obligatorio, alérgenos y estado de disponibilidad.
- **FR-003**: El sistema DEBE gestionar menús en borrador, invisibles para los clientes, y permitir previsualizarlos.
- **FR-004**: El sistema DEBE bloquear la publicación de productos sin texto alternativo o sin información de alérgenos.
- **FR-005**: Cada publicación DEBE crear una versión nueva e inmutable del menú; el sistema conserva el historial con autor, fecha y cambios.
- **FR-006**: Los cambios publicados DEBEN ser visibles para los clientes en menos de 10 segundos, sin desplegar la aplicación.
- **FR-007**: El sistema DEBE permitir definir horarios de disponibilidad por menú (días y franjas horarias).
- **FR-008**: El sistema DEBE permitir gestionar la disponibilidad por producto: disponible, agotado y, opcionalmente, existencias que se descuentan con cada venta confirmada.
- **FR-009**: El personal DEBE poder cambiar la disponibilidad y las existencias, pero no precios ni contenidos.
- **FR-010**: Los clientes DEBEN poder navegar por categorías y filtrar por alérgenos a excluir.
- **FR-011**: El sistema DEBE detectar ediciones concurrentes del mismo menú y evitar que se pierdan cambios.
- **FR-012**: Todos los cambios de precios y contenidos DEBEN quedar en el registro de auditoría.
- **FR-013**: La tienda y la administración del menú DEBEN cumplir WCAG 2.2 AAA, incluida la indicación de disponibilidad con texto y no solo con color.

### Key Entities

- **Menú**: nombre, horario de vigencia, estado (borrador, publicado, retirado) y versiones.
- **Versión de menú**: instantánea inmutable de categorías, productos y precios en una publicación.
- **Categoría**: agrupa productos dentro de un menú, con un orden.
- **Producto**: nombre, descripción, precio, imagen, texto alternativo, alérgenos, disponibilidad y existencias.
- **Alérgeno**: lista cerrada de alérgenos de declaración obligatoria.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 95 % de las cargas del menú muestran el contenido en menos de 1 segundo, también durante los picos de festividades.
- **SC-002**: Un cambio publicado es visible para los clientes en menos de 10 segundos.
- **SC-003**: Un administrador crea y publica un menú de 20 productos en menos de 15 minutos.
- **SC-004**: El 100 % de los pedidos conserva el precio y la descripción vigentes al comprar, aunque el menú cambie después.
- **SC-005**: El 100 % de los productos publicados tiene texto alternativo e información de alérgenos.

## Assumptions

- Hay una sola cafetería, con una sola zona horaria; varias sucursales quedan fuera de la primera versión.
- El menú se muestra en español; el multilenguaje queda fuera de la primera versión.
- Las existencias son opcionales por producto; muchos productos de cafetería se preparan al momento y solo se marcan como agotados.
- La lista de alérgenos sigue la normativa de etiquetado vigente en Ecuador.
- Las promociones y descuentos se gestionan en el spec 005; este spec solo cubre el precio base.
