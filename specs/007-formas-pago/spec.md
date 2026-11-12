# Feature Specification: Formas de pago

**Feature Branch**: `007-formas-pago`
**Created**: 2026-11-12
**Status**: Draft
**Input**: User description: "RF-05 Formas de pago: tarjeta de crédito mediante una pasarela externa con tokenización (el sistema nunca recibe ni almacena el número de tarjeta); transferencia, que queda pendiente hasta que el personal o la conciliación bancaria confirma el pago; efectivo, registrado al entregar. La confirmación de pagos es idempotente."

## User Scenarios & Testing *(mandatory)*

Actores: **cliente**, **personal de cafetería** y **pasarela de pago** (sistema externo).

### User Story 1 - Pagar con tarjeta (Priority: P1)

El cliente elige tarjeta, introduce sus datos en el formulario seguro de la pasarela y, al aprobarse el pago, su pedido pasa a *pagado* y entra en la cola de preparación.

**Why this priority**: es la forma de pago principal en una tienda online y la de mayor riesgo.

**Independent Test**: en el entorno de pruebas de la pasarela, pagar un pedido con una tarjeta de prueba aprobada y otra rechazada.

**Acceptance Scenarios**:

1. **Dado** un pedido pendiente de pago, **Cuando** el cliente paga con una tarjeta aprobada, **Entonces** el pedido pasa a *pagado* y el cliente ve la confirmación.
2. **Dado** una tarjeta rechazada, **Cuando** se intenta el pago, **Entonces** el cliente ve un mensaje claro y puede reintentar o elegir otra forma de pago, sin perder el pedido.
3. **Dado** que la pasarela notifica el mismo pago dos veces, **Cuando** el sistema recibe las notificaciones, **Entonces** el pedido se confirma una sola vez y se cobra una sola vez.
4. **Dado** que el cliente cierra la página tras pagar, **Cuando** la pasarela confirma el pago, **Entonces** el pedido pasa a *pagado* igualmente.
5. **Dado** un pago con tarjeta, **Cuando** se procesa, **Entonces** ningún componente de la cafetería recibe ni guarda el número completo de la tarjeta ni su código de seguridad.

---

### User Story 2 - Pagar en efectivo (Priority: P1)

El cliente elige efectivo; el pedido se prepara y el personal registra el cobro al entregarlo.

**Why this priority**: forma de pago habitual en cafeterías y la más simple de implementar.

**Independent Test**: hacer un pedido en efectivo, prepararlo y registrar el cobro al entregarlo.

**Acceptance Scenarios**:

1. **Dado** un cliente que elige efectivo, **Cuando** confirma, **Entonces** el pedido entra en la cola de preparación como *pago en efectivo pendiente*.
2. **Dado** un pedido en efectivo listo, **Cuando** el personal lo entrega y registra el cobro con el importe recibido, **Entonces** el pedido pasa a *entregado* y pagado, y se muestra el cambio a devolver.
3. **Dado** un cliente con varios pedidos en efectivo sin retirar, **Cuando** intenta un nuevo pedido en efectivo, **Entonces** se aplica el límite definido por el administrador.

---

### User Story 3 - Pagar por transferencia (Priority: P2)

El cliente elige transferencia, recibe los datos bancarios y una referencia única, hace la transferencia y adjunta el comprobante. El personal verifica el pago y el pedido pasa a *pagado*.

**Why this priority**: requisito del negocio, pero con más fricción y menos uso esperado que tarjeta y efectivo.

**Independent Test**: hacer un pedido por transferencia, adjuntar un comprobante y que el personal lo confirme.

**Acceptance Scenarios**:

1. **Dado** un cliente que elige transferencia, **Cuando** confirma, **Entonces** ve los datos bancarios, el importe exacto y una referencia única del pedido.
2. **Dado** un cliente que adjunta su comprobante, **Cuando** el personal verifica que el importe ha llegado, **Entonces** confirma el pago y el pedido pasa a *pagado*.
3. **Dado** un comprobante que no corresponde, **Cuando** el personal lo rechaza con un motivo, **Entonces** el cliente es notificado y puede adjuntar otro o cambiar de forma de pago.
4. **Dado** un pedido por transferencia, **Cuando** pasa el plazo de pago sin confirmación, **Entonces** se cancela automáticamente (spec 006).

---

### User Story 4 - Reembolsos (Priority: P2)

Cuando se cancela un pedido pagado, el cliente recupera su dinero por la misma vía con la que pagó.

**Why this priority**: obligatorio para operar con cancelaciones, aunque sea poco frecuente.

**Independent Test**: cancelar un pedido pagado con tarjeta y otro pagado por transferencia y verificar que se inicia el reembolso correspondiente.

**Acceptance Scenarios**:

1. **Dado** un pedido pagado con tarjeta que se cancela, **Cuando** se inicia el reembolso, **Entonces** se solicita a la pasarela y el cliente recibe la confirmación cuando se completa.
2. **Dado** un pedido pagado por transferencia que se cancela, **Cuando** se inicia el reembolso, **Entonces** queda una tarea para el personal con los datos necesarios, y el cliente es informado del plazo.
3. **Dado** un reembolso, **Cuando** se solicita dos veces por un reintento, **Entonces** se devuelve el dinero una sola vez.

### Edge Cases

- La pasarela no responde: el cliente ve un mensaje claro; el pedido sigue pendiente y el sistema comprueba más tarde el estado real del pago con la pasarela.
- La pasarela confirma un pago para un pedido ya cancelado: se reembolsa automáticamente.
- Pago parcial o importe distinto en una transferencia: no se confirma; el personal lo gestiona manualmente.
- Notificaciones de la pasarela falsificadas: se rechazan las que no estén firmadas por la pasarela.
- El cliente intenta pagar un pedido ya pagado: se le muestra que ya está pagado y no se cobra de nuevo.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE ofrecer tres formas de pago: tarjeta de crédito, transferencia y efectivo; el administrador puede activar o desactivar cada una.
- **FR-002**: El pago con tarjeta DEBE hacerse en el formulario seguro de la pasarela; el sistema NUNCA recibe ni almacena el número completo de la tarjeta ni su código de seguridad.
- **FR-003**: El sistema DEBE confirmar los pagos con tarjeta a partir de la notificación verificada de la pasarela, no solo por la redirección del navegador.
- **FR-004**: La confirmación de cualquier pago y cualquier reembolso DEBE ser idempotente: notificaciones o reintentos duplicados no generan cobros, confirmaciones ni reembolsos duplicados.
- **FR-005**: El sistema DEBE rechazar las notificaciones de pago que no estén firmadas por la pasarela.
- **FR-006**: El sistema DEBE consultar periódicamente el estado de los pagos con tarjeta sin confirmar para resolver notificaciones perdidas.
- **FR-007**: Para transferencias, el sistema DEBE mostrar los datos bancarios, el importe y una referencia única, y permitir adjuntar un comprobante.
- **FR-008**: El personal DEBE poder confirmar o rechazar transferencias con motivo, y registrar cobros en efectivo con el importe recibido.
- **FR-009**: El sistema DEBE gestionar reembolsos por la misma vía del pago: automáticos para tarjeta y como tarea para el personal en transferencias.
- **FR-010**: El sistema DEBE permitir al administrador limitar los pedidos en efectivo pendientes de retirar por cliente.
- **FR-011**: Cada operación de pago (intento, confirmación, rechazo, reembolso) DEBE quedar en un registro de auditoría inmutable.
- **FR-012**: El sistema DEBE proteger la integración con la pasarela frente a lentitud o fallos (tiempos de espera, reintentos y corte temporal), sin bloquear el resto de la tienda.
- **FR-013**: Las pantallas de pago DEBEN cumplir WCAG 2.2 AAA, sin límites de tiempo que el cliente no pueda ampliar.

### Key Entities

- **Pago**: pedido, forma de pago, importe, estado (pendiente, aprobado, rechazado, reembolsado) y referencia externa.
- **Notificación de la pasarela**: identificador único, tipo, fecha y resultado del procesamiento.
- **Comprobante de transferencia**: archivo adjunto, referencia del pedido, estado de verificación y verificador.
- **Reembolso**: pago de origen, importe, estado y fecha.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 95 % de los pagos con tarjeta aprobados se reflejan como *pagado* en menos de 10 segundos.
- **SC-002**: 0 cobros o reembolsos duplicados en las pruebas de notificaciones repetidas y de fallos.
- **SC-003**: 0 números de tarjeta en los sistemas, registros o copias de seguridad de la cafetería (verificado por auditoría).
- **SC-004**: El personal confirma una transferencia en menos de 1 minuto con el comprobante a la vista.
- **SC-005**: Una caída de la pasarela no afecta a la navegación del menú ni a los pagos en efectivo y por transferencia.

## Assumptions

- La pasarela elegida (pendiente en `ROADMAP.md`) ofrece formulario alojado o tokenización, notificaciones firmadas, consulta de estado, reembolsos y entorno de pruebas.
- La confirmación de transferencias es manual en la primera versión; la conciliación automática con el banco queda para una versión posterior.
- El cobro en efectivo se hace al retirar el pedido en la cafetería; no hay entrega a domicilio en la primera versión (aclaración del spec 006).
- Los comprobantes de transferencia contienen datos personales y se eliminan tras el plazo de retención definido (LOPDP).
- La moneda es el dólar estadounidense.
