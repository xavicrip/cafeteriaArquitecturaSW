# Feature Specification: Identidad y gestión de clientes

**Feature Branch**: `002-identidad-clientes`
**Created**: 2026-11-12
**Status**: Draft
**Input**: User description: "RF-01 Gestión de clientes: registro, inicio de sesión, recuperación de contraseña, roles de cliente, personal y administrador con MFA para personal y administradores, perfil del cliente, historial de pedidos y derechos de protección de datos (LOPDP) para ver, editar, exportar y eliminar los datos personales."

## User Scenarios & Testing *(mandatory)*

Actores: **cliente**, **personal de cafetería** y **administrador**.

### User Story 1 - Crear una cuenta e iniciar sesión (Priority: P1)

Una persona se registra con su nombre, correo y contraseña, acepta la política de privacidad, confirma su correo e inicia sesión. Puede cerrar sesión y recuperar el acceso si olvida su contraseña.

**Why this priority**: el carrito, los pedidos y los pagos están asociados a un cliente; sin identidad no hay compra.

**Independent Test**: registrarse, confirmar el correo, iniciar y cerrar sesión, y recuperar la contraseña, todo solo con teclado y con lector de pantalla.

**Acceptance Scenarios**:

1. **Dado** una persona sin cuenta, **Cuando** completa el registro con datos válidos y acepta la política de privacidad, **Entonces** se crea la cuenta y recibe un correo de confirmación.
2. **Dado** un correo ya registrado, **Cuando** alguien intenta registrarse con él, **Entonces** el sistema no revela si la cuenta existe y envía al titular un aviso por correo.
3. **Dado** un cliente con correo confirmado, **Cuando** introduce credenciales correctas, **Entonces** inicia sesión.
4. **Dado** varios intentos fallidos de inicio de sesión, **Cuando** se supera el límite, **Entonces** el sistema frena nuevos intentos temporalmente y avisa al titular.
5. **Dado** un cliente que olvidó su contraseña, **Cuando** solicita recuperarla, **Entonces** recibe un enlace de un solo uso con caducidad y puede definir una nueva contraseña.
6. **Dado** un error en el formulario, **Cuando** se envía, **Entonces** el mensaje indica qué campo falla y cómo corregirlo, y el foco va al primer error.

---

### User Story 2 - Acceso del personal y administradores con doble factor (Priority: P1)

El administrador crea cuentas de personal. El personal y los administradores inician sesión con contraseña y un segundo factor, y solo ven las funciones de su rol.

**Why this priority**: el personal confirma pagos y gestiona pedidos; un acceso indebido compromete el dinero y los datos.

**Independent Test**: crear una cuenta de personal, configurar el segundo factor, iniciar sesión y comprobar que no puede acceder a funciones de administrador.

**Acceptance Scenarios**:

1. **Dado** un administrador, **Cuando** crea una cuenta de personal, **Entonces** la persona recibe una invitación y debe configurar el segundo factor en su primer acceso.
2. **Dado** un miembro del personal sin segundo factor configurado, **Cuando** intenta usar funciones del personal, **Entonces** se le exige configurarlo antes.
3. **Dado** un miembro del personal, **Cuando** intenta una función de administrador, **Entonces** se le deniega y el intento queda registrado.
4. **Dado** un administrador que desactiva una cuenta de personal, **Cuando** se desactiva, **Entonces** sus sesiones activas terminan de inmediato.

---

### User Story 3 - Gestionar el perfil y ver el historial (Priority: P2)

El cliente consulta y edita sus datos (nombre, teléfono, preferencias), cambia su contraseña y ve el historial de sus pedidos.

**Why this priority**: mejora la experiencia, pero la compra es posible sin él.

**Independent Test**: editar el perfil y ver la lista de pedidos de un cliente con pedidos previos.

**Acceptance Scenarios**:

1. **Dado** un cliente con sesión iniciada, **Cuando** edita sus datos, **Entonces** se guardan y se confirma el cambio de forma accesible.
2. **Dado** un cliente que cambia su correo, **Cuando** lo cambia, **Entonces** debe confirmar el nuevo correo y se avisa al anterior.
3. **Dado** un cliente con pedidos, **Cuando** abre su historial, **Entonces** ve sus pedidos ordenados del más reciente al más antiguo con fecha, total y estado.

---

### User Story 4 - Ejercer los derechos sobre mis datos (Priority: P2)

El cliente puede ver qué datos personales guarda la cafetería y para qué, gestionar sus consentimientos, descargar una copia de sus datos y eliminar su cuenta.

**Why this priority**: obligación legal (LOPDP) antes del lanzamiento con datos reales.

**Independent Test**: un cliente descarga sus datos, retira el consentimiento de marketing y elimina su cuenta; después no puede iniciar sesión y sus pedidos aparecen anonimizados en los reportes.

**Acceptance Scenarios**:

1. **Dado** un cliente, **Cuando** solicita una copia de sus datos, **Entonces** recibe un archivo legible por máquina con todos sus datos personales y pedidos.
2. **Dado** un cliente, **Cuando** retira un consentimiento opcional, **Entonces** el tratamiento asociado se detiene y el cambio queda registrado con fecha.
3. **Dado** un cliente, **Cuando** confirma la eliminación de su cuenta (con nueva autenticación), **Entonces** sus datos personales se eliminan, sus pedidos se anonimizan y recibe confirmación.
4. **Dado** un cliente con un pedido en curso, **Cuando** solicita eliminar su cuenta, **Entonces** se le informa de que la eliminación se completará al terminar ese pedido.

### Edge Cases

- El enlace de confirmación o de recuperación ya se usó o caducó: se informa y se ofrece enviar uno nuevo.
- Un cliente intenta iniciar sesión sin haber confirmado su correo: se le explica y se ofrece reenviar la confirmación.
- El personal pierde su segundo factor: un administrador puede restablecerlo tras verificar la identidad; la acción queda registrada.
- Contraseñas débiles o filtradas en brechas conocidas: se rechazan con una explicación.
- Sesión inactiva durante mucho tiempo: caduca, pero el carrito se conserva (ver spec 004) y el cliente vuelve al punto en que estaba tras iniciar sesión.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE permitir el registro con nombre, correo y contraseña, y la aceptación explícita de la política de privacidad.
- **FR-002**: El sistema DEBE confirmar el correo antes de permitir comprar.
- **FR-003**: El sistema DEBE permitir iniciar y cerrar sesión, y recuperar la contraseña mediante un enlace de un solo uso que caduca en 30 minutos.
- **FR-004**: El sistema DEBE exigir contraseñas de al menos 12 caracteres y rechazar contraseñas filtradas en brechas conocidas.
- **FR-005**: El sistema DEBE limitar los intentos de inicio de sesión y de registro para frenar ataques automatizados.
- **FR-006**: El sistema NO DEBE revelar si un correo está registrado en las respuestas de registro, inicio de sesión ni recuperación.
- **FR-007**: El sistema DEBE soportar los roles cliente, personal y administrador, y comprobar el rol en el servidor en cada operación.
- **FR-008**: El sistema DEBE exigir un segundo factor de autenticación al personal y a los administradores.
- **FR-009**: Los administradores DEBEN poder crear, desactivar y reactivar cuentas de personal; desactivar una cuenta termina sus sesiones.
- **FR-010**: El cliente DEBE poder ver y editar sus datos de perfil y cambiar su contraseña; cambiar el correo o la contraseña requiere confirmar la identidad.
- **FR-011**: El cliente DEBE poder ver su historial de pedidos.
- **FR-012**: El sistema DEBE mostrar al cliente qué datos personales se guardan, con qué finalidad y durante cuánto tiempo.
- **FR-013**: El cliente DEBE poder dar y retirar consentimientos opcionales; el sistema registra cada cambio con fecha.
- **FR-014**: El cliente DEBE poder descargar todos sus datos personales en un formato estructurado y legible por máquina.
- **FR-015**: El cliente DEBE poder eliminar su cuenta; el sistema elimina sus datos personales y anonimiza sus pedidos en un plazo máximo de 15 días (límite legal) y, como objetivo, de inmediato.
- **FR-016**: El sistema DEBE registrar los eventos de seguridad (inicios de sesión, fallos, cambios de rol, restablecimientos de segundo factor) sin incluir contraseñas ni datos sensibles.
- **FR-017**: Todas las pantallas de esta funcionalidad DEBEN cumplir WCAG 2.2 AAA.

### Key Entities

- **Cuenta**: credenciales, estado (pendiente de confirmar, activa, desactivada), rol y segundo factor.
- **Cliente**: perfil con datos personales (nombre, correo, teléfono, preferencias); se relaciona con sus pedidos y su carrito.
- **Consentimiento**: finalidad, estado y fecha de cada cambio.
- **Solicitud de derechos**: tipo (acceso, rectificación, eliminación, oposición, portabilidad), fecha y estado.
- **Evento de seguridad**: tipo, cuenta, fecha y origen.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Un nuevo cliente completa el registro en menos de 2 minutos.
- **SC-002**: El 95 % de los clientes que inician el registro lo completan al primer intento sin errores bloqueantes.
- **SC-003**: El 100 % de los flujos de esta funcionalidad se puede completar solo con teclado y con lector de pantalla.
- **SC-004**: El 100 % de las cuentas de personal y administración tiene segundo factor activo.
- **SC-005**: La descarga de datos personales está disponible en menos de 1 minuto tras solicitarla.
- **SC-006**: Tras eliminar una cuenta, ningún dato personal del cliente aparece en búsquedas, reportes ni exportaciones.

## Assumptions

- El inicio de sesión es con correo y contraseña; el inicio con redes sociales queda fuera de la primera versión.
- Los clientes deben registrarse para comprar; no hay compra como invitado en la primera versión.
- El segundo factor del personal es una aplicación de autenticación (códigos temporales); los clientes pueden activarlo de forma opcional.
- La ley aplicable es la LOPDP de Ecuador (pendiente de confirmar en `ROADMAP.md`); el plazo legal para atender solicitudes de derechos es de 15 días.
- Los pedidos anonimizados se conservan para la contabilidad durante el plazo que exija la normativa tributaria.
