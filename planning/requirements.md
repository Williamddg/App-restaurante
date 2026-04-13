# Requisitos para Sistema de Gestión de Restaurante

*Aplicación web para restaurantes pequeños y medianos, con operación en tiempo real, despliegue Local y Nube, y arquitectura mantenible lista para operación real.*

---

## 1. Introducción y objetivos

### 1.1 Propósito del documento

Definir de forma ejecutable los requisitos funcionales, no funcionales, arquitectura, modelo de datos, contratos API, seguridad, operación y pruebas del sistema.

### 1.2 Objetivo del producto

El sistema debe cubrir operación end-to-end para:

* Cliente (kiosco)
* Mesero
* Cocina
* Caja
* Administrador

### 1.3 Modos de despliegue

1. **Modo Local:** aplicación y base de datos en infraestructura del restaurante.
2. **Modo Nube:** aplicación en infraestructura del proveedor.

La experiencia funcional debe ser equivalente en ambos modos.

### 1.4 Decisión arquitectónica obligatoria (Nube / Multi-tenant)

En modo Nube se adopta esta decisión, sin excepciones para la versión base:

* Existe **una única aplicación compartida** (frontend + backend) para múltiples restaurantes.
* Cada restaurante (tenant) tiene **su propia base de datos separada**.
* El sistema es multi-tenant a nivel de lógica y configuración, pero **no** comparte base operativa entre restaurantes.
* Cada tenant puede personalizar: logo, colores, razón social, nombre comercial, datos fiscales, textos legales y parámetros operativos.

### 1.5 Objetivos de negocio

* Reducir tiempos de atención.
* Evitar errores manuales en pedidos/cobros.
* Unificar cocina, mesero, caja y administración.
* Comercializar en modelo Licencia Local y SaaS Nube.

### 1.6 Alcance de versión base (obligatorio)

* Mesas, cuentas y pedidos.
* Cocina en tiempo real.
* Cobro completo con propina y pagos mixtos.
* Factura PDF e impresión.
* Descuentos configurables.
* División de cuenta por cliente.
* Inventario manual.
* Reportes operativos.
* Seguridad base temprana.
* Operación/backup/restore/update/rollback.

### 1.7 Fuera de alcance de la versión base

* Integración bancaria directa con pasarelas.
* Facturación fiscal certificada gubernamental.
* Reversión/anulación de cobro ya facturado (queda pendiente de definición formal).
* Gestión de agotados temporales por horario/cocina (futuro opcional).

---

## 2. Actores, roles y permisos

### 2.1 Roles

| Rol | Acceso | Capacidades principales |
|---|---|---|
| Cliente (kiosco) | Interfaz kiosco autenticada por usuario kiosco | Crear pedido nuevo para mesa libre |
| Mesero | Tablet/móvil | Gestionar mesas, agregar ítems, dividir cuenta por cliente |
| Cocina | Pantalla cocina | Tomar ítems, marcar en preparación/listo |
| Caja | Mostrador | Cobrar, facturar, imprimir, reenviar factura |
| Administrador | Web admin | Configuración total, catálogos, usuarios, reportes, operación |

### 2.2 Reglas clave de autorización

* RBAC en backend y frontend.
* Todas las acciones auditables validan usuario + permisos + tenant.
* En Nube no puede existir acceso cruzado entre tenants.
* Al desactivar usuario, se invalidan sesiones activas.

---

## 3. Requisitos funcionales

### 3.1 Requisitos globales

* **RF-GEN-001:** Misma funcionalidad en Local y Nube.
* **RF-GEN-002:** Personalización por tenant (branding + datos fiscales + configuración).
* **RF-GEN-003:** Aislamiento total de datos por tenant (BD separada por tenant en Nube).
* **RF-GEN-004:** Auditoría obligatoria de acciones críticas.

### 3.2 Autenticación y sesiones

* **RF-AUT-001:** Login por email + contraseña.
* **RF-AUT-002:** Cookies `HttpOnly`, `Secure` (en HTTPS), `SameSite` configurado.
* **RF-AUT-003:** Expiración por inactividad (default 30 min).
* **RF-AUT-004:** Expiración absoluta (default 8 h).
* **RF-AUT-005:** Rate limiting de login.
* **RF-AUT-006:** Registro de intentos fallidos.
* **RF-AUT-007:** Invalidación de sesiones al desactivar usuario.

### 3.3 Mesas y cuentas

* **RF-MES-001:** Los números de mesa son únicos y **no se reordenan manualmente**.
* **RF-MES-002:** Se pueden agregar nuevas mesas sin afectar numeración existente.
* **RF-MES-003:** Estados de mesa: `libre`, `ocupada`, `pagando`, `deshabilitada`.
* **RF-MES-004:** Una sola cuenta activa por mesa.
* **RF-MES-005:** Primer pedido abre cuenta automáticamente si mesa está libre.
* **RF-MES-006:** Estados de cuenta: `abierta`, `pagando`, `cerrada`, `cancelada`.
* **RF-MES-007:** Cuenta en `pagando` o `cerrada` no acepta nuevos ítems.

### 3.4 Pedidos y catálogo

* **RF-PED-001:** Pedido puede originarse por kiosco, mesero o caja (si flag activa).
* **RF-PED-002:** Ítem de cocina es unidad operativa (no solo ticket completo).
* **RF-PED-003:** Estado de ítem: `pendiente`, `en_preparacion`, `listo`, `entregado`, `cancelado`.
* **RF-PED-004:** Solo se cancela ítem en `pendiente`.
* **RF-PED-005:** Se guarda snapshot de nombre, precio y medida al momento de venta.

### 3.5 Extras y tamaños/medidas

* **RF-PRD-001:** Los extras se modelan como productos/platos independientes (no como campo libre).
* **RF-PRD-002:** Todo ítem vendible tiene unidad de medida: `unidad`, `g`, `kg`, `ml`, `l` (extensible).
* **RF-PRD-003:** Cocina visualiza: producto, medida, cantidad, mesa y observaciones.

### 3.6 Cocina con múltiples pantallas

* **RF-COC-001:** Varias pantallas de cocina consumen la misma cola base.
* **RF-COC-002:** Cada persona/sección identifica manualmente qué productos prepara.
* **RF-COC-003:** No se requiere partición técnica avanzada por estación en esta etapa.

### 3.7 Descuentos

* **RF-DCT-001:** Soporte a descuentos por porcentaje.
* **RF-DCT-002:** Soporte a descuentos por valor fijo.
* **RF-DCT-003:** Vigencia por rango de fechas.
* **RF-DCT-004:** Restricción por horario (ej. 14:00–17:00).
* **RF-DCT-005:** Monto mínimo de cuenta para aplicar.
* **RF-DCT-006:** Reglas configurables por administración.
* **RF-DCT-007:** Registrar descuento aplicado en cuenta/factura/reporte/auditoría.

### 3.8 Cobros, propina y caja

* **RF-CAJ-001:** Soporte de propina en cobro.
* **RF-CAJ-002:** Soporte de pagos mixtos en una misma transacción de cierre.
* **RF-CAJ-003:** Si pago incluye efectivo, calcular y mostrar vuelto.
* **RF-CAJ-004:** No se permiten pagos parciales dejando saldo pendiente abierto.
* **RF-CAJ-005:** Sí se permite cubrir el total usando varios métodos en la misma transacción.
* **RF-CAJ-006:** No se cierra cuenta si existen ítems en `en_preparacion`.

### 3.9 División de cuenta y clientes

* **RF-CLI-001:** División de cuenta por cliente (subcuentas dentro de la cuenta principal).
* **RF-CLI-002:** Cada ítem puede asociarse a cliente de mesa (opcional).
* **RF-CLI-003:** Caja puede cobrar por subcuenta o total consolidado.
* **RF-CLI-004:** Reportes administrativos permiten filtros por cliente.

### 3.10 Factura, impresión y envío

* **RF-FAC-001:** Cierre exitoso genera comprobante/factura PDF.
* **RF-FAC-002:** Numeración consecutiva por tenant.
* **RF-FAC-003:** Botón de impresión (ticket 80mm o A4 configurable).
* **RF-FAC-004:** Reenvío por email y trazabilidad del resultado.

### 3.11 Reportes

* **RF-REP-001:** Ventas por fecha, método, origen y usuario.
* **RF-REP-002:** Descuentos aplicados (tipo, monto, regla).
* **RF-REP-003:** Propinas por período y por cajero.
* **RF-REP-004:** Ventas por cliente (cuando haya asociación).
* **RF-REP-005:** Exportación CSV/PDF.

### 3.12 Operación y despliegue

* **RF-OPS-001:** Alta/provisión de tenant en Nube con BD dedicada y branding inicial.
* **RF-OPS-002:** Startup checks: BD accesible, migraciones aplicadas, configuración mínima, licencia/suscripción válida.
* **RF-OPS-003:** Health checks: `/health/live`, `/health/ready`.
* **RF-OPS-004:** Backups programados y manuales.
* **RF-OPS-005:** Restore documentado y probado.
* **RF-OPS-006:** Actualización con backup previo y rollback.

---

## 4. Contratos API mínimos (request/response)

> Los errores se devuelven en formato estándar `{ "error": { "code": "...", "message": "..." } }`.

### 4.1 Autenticación

#### `POST /api/auth/login`
**Request**
```json
{ "email": "admin@demo.com", "password": "secret" }
```
**Response 200**
```json
{
  "user": { "id": "u1", "name": "Admin", "email": "admin@demo.com", "role": "admin" },
  "session": { "idle_expires_at": "2026-04-13T12:00:00Z", "absolute_expires_at": "2026-04-13T20:00:00Z" }
}
```

#### `POST /api/auth/logout`
**Request:** sin body.
**Response 200**
```json
{ "ok": true }
```

#### `GET /api/auth/me`
**Response 200**
```json
{ "user": { "id": "u1", "name": "Admin", "role": "admin", "tenant_id": "t1" } }
```

### 4.2 Mesas y cuentas

#### `POST /api/tables`
```json
{ "table_number": 21 }
```
**Response 201**
```json
{ "id": "tb21", "table_number": 21, "status": "libre", "enabled": true }
```

#### `GET /api/tables/status`
**Response 200**
```json
{
  "items": [
    { "table_id": "tb1", "table_number": 1, "status": "ocupada", "account_id": "acc1" }
  ]
}
```

#### `POST /api/accounts/open`
```json
{ "table_id": "tb1" }
```
**Response 201**
```json
{ "account_id": "acc1", "status": "abierta" }
```

### 4.3 Pedidos

#### `POST /api/orders`
```json
{
  "table_id": "tb1",
  "origin": "mesero",
  "idempotency_key": "f9f1-...",
  "items": [
    { "dish_id": "d_bife_300g", "quantity": 2, "notes": "término medio" }
  ]
}
```
**Response 201**
```json
{
  "ticket_id": "ot1",
  "account_id": "acc1",
  "items": [
    { "item_id": "oi1", "status": "pendiente", "dish_name": "Bife 300g", "measure": "unidad" }
  ]
}
```

#### `PATCH /api/orders/items/{item_id}/cancel`
```json
{ "reason": "error de carga" }
```
**Response 200**
```json
{ "item_id": "oi1", "status": "cancelado" }
```

### 4.4 Cocina

#### `GET /api/kitchen/queue`
**Response 200**
```json
{
  "items": [
    {
      "item_id": "oi1",
      "table_number": 4,
      "dish_name": "Papas fritas",
      "measure": "unidad",
      "quantity": 1,
      "status": "pendiente",
      "created_at": "2026-04-13T10:00:00Z"
    }
  ]
}
```

#### `PATCH /api/kitchen/items/{item_id}/status`
```json
{ "status": "en_preparacion" }
```
**Response 200**
```json
{ "item_id": "oi1", "status": "en_preparacion" }
```

### 4.5 Descuentos

#### `POST /api/admin/discount-rules`
```json
{
  "name": "Happy Hour",
  "type": "percentage",
  "value": 15,
  "date_start": "2026-05-01",
  "date_end": "2026-05-31",
  "time_start": "14:00",
  "time_end": "17:00",
  "min_amount": 20
}
```
**Response 201**
```json
{ "rule_id": "dr1", "active": true }
```

### 4.6 Cobro

#### `POST /api/cash/accounts/{account_id}/checkout`
```json
{
  "tip_amount": 4.50,
  "discounts": [{ "rule_id": "dr1" }],
  "payments": [
    { "method": "cash", "amount": 30 },
    { "method": "card", "amount": 12.5 }
  ],
  "customer_email": "cliente@mail.com"
}
```
**Reglas:** suma de `payments` debe cubrir exactamente `total_final`; no se permite saldo pendiente.

**Response 200**
```json
{
  "account_id": "acc1",
  "status": "cerrada",
  "amounts": {
    "subtotal": 35,
    "discount_total": 3,
    "tip_amount": 4.5,
    "total_final": 36.5,
    "paid_total": 42.5,
    "change_amount": 6
  },
  "invoice": { "invoice_id": "inv1", "invoice_number": "A-00001234", "pdf_url": "/files/invoices/2026/04/13/A-00001234.pdf" }
}
```

### 4.7 División de cuenta por cliente

#### `POST /api/accounts/{account_id}/customers`
```json
{ "name": "Cliente 1" }
```
**Response 201**
```json
{ "customer_id": "ac1", "name": "Cliente 1" }
```

#### `PATCH /api/orders/items/{item_id}/assign-customer`
```json
{ "customer_id": "ac1" }
```
**Response 200**
```json
{ "item_id": "oi1", "customer_id": "ac1" }
```

---

## 5. Requisitos no funcionales

### 5.1 Rendimiento objetivo

* Consultas frecuentes < 2s.
* Escrituras críticas < 2.5s.
* Propagación tiempo real < 3s.

### 5.2 Seguridad temprana obligatoria

* Rate limiting de login desde Sprint 0.
* Cookies seguras y sesiones endurecidas desde Sprint 0.
* Validaciones de entrada desde Sprint 0.
* Auditoría de autenticación desde Sprint 0.

### 5.3 Observabilidad

* Logs técnicos + negocio.
* Correlation/request id.
* Health y readiness checks.
* Startup checks con fail-fast.

### 5.4 Mantenibilidad

* Código modular por dominio.
* Versionado semántico.
* Deploy reproducible con Docker Compose.

---

## 6. Modelo de datos

### 6.1 Entidades principales

#### 6.1.1 Plataforma/tenant

* `tenants` (id, code, commercial_name, legal_name, tax_id, active)
* `tenant_branding` (tenant_id, logo_url, primary_color, secondary_color, invoice_footer)
* `tenant_settings` (tenant_id, cash_can_create_orders, kiosk_enabled, timezone, currency)
* `tenant_databases` (tenant_id, db_host, db_name, db_user_ref, status)

#### 6.1.2 Seguridad

* `users` (id, tenant_id, email, password_hash, full_name, active, last_login_at)
* `roles`, `permissions`, `user_roles`, `role_permissions`
* `user_sessions` (id, user_id, created_at, idle_expires_at, absolute_expires_at, revoked_at)
* `auth_attempts` (id, tenant_id, email, success, ip, created_at)

#### 6.1.3 Operación restaurante

* `tables` (id, tenant_id, table_number, enabled, created_at)
* `table_accounts` (id, tenant_id, table_id, status, subtotal_amount, discount_amount, tip_amount, total_amount, version)
* `account_customers` (id, tenant_id, table_account_id, name)
* `order_tickets` (id, tenant_id, table_account_id, origin, created_by_user_id, created_at)
* `order_items` (id, tenant_id, order_ticket_id, dish_id, dish_name_snapshot, measure_snapshot, unit_price_snapshot, quantity, status, account_customer_id nullable)

#### 6.1.4 Catálogo y descuentos

* `menu_categories` (id, tenant_id, name, active)
* `dishes` (id, tenant_id, category_id, name, description, measure_unit, price, active)
* `discount_rules` (id, tenant_id, name, type[`percentage|fixed`], value, date_start, date_end, time_start, time_end, min_amount, active)
* `account_discounts` (id, tenant_id, table_account_id, discount_rule_id nullable, discount_name_snapshot, amount)

#### 6.1.5 Cobro y facturación

* `payment_methods` (id, tenant_id, code, name, active)
* `payments` (id, tenant_id, table_account_id, method_id, amount, reference, received_by_user_id)
* `account_checkouts` (id, tenant_id, table_account_id, subtotal, discount_total, tip_amount, total_final, paid_total, change_amount, closed_by_user_id, closed_at)
* `invoice_sequences` (tenant_id, current_value)
* `invoices` (id, tenant_id, table_account_id, invoice_number, pdf_path, customer_email, email_status, issued_by_user_id)

#### 6.1.6 Inventario y auditoría

* `inventory_items`, `inventory_lots`, `inventory_movements`
* `audit_logs` (tenant_id, user_id, action, entity, before_json, after_json, created_at)

### 6.2 Restricciones mínimas

* `tables(tenant_id, table_number)` único.
* No reuso de número de mesa para otra mesa activa.
* Una sola cuenta activa por mesa.
* `invoices(tenant_id, invoice_number)` único.
* Todas las tablas operativas incluyen `tenant_id` (en Nube, además cada tenant tiene su BD dedicada).

---

## 7. Arquitectura y operación (Local/Nube)

### 7.1 Arquitectura Nube

* App compartida (API + frontend + realtime).
* Resolución de tenant por subdominio/cabecera/login.
* Conector dinámico a base dedicada por tenant.
* Control plane separado para aprovisionamiento, suscripción y catálogo de releases.

### 7.2 Aprovisionamiento de tenant

1. Alta comercial del tenant.
2. Creación de base de datos dedicada.
3. Ejecución de esquema/seed.
4. Carga de branding y settings base.
5. Creación de admin tenant.
6. Activación de suscripción y chequeo final.

### 7.3 Startup checks obligatorios

* Conectividad base de datos tenant.
* Existencia de tablas mínimas.
* Carga de settings obligatorios.
* Estado de licencia/suscripción.
* Permisos de storage para facturas/backups.

### 7.4 Backups, restore, update, rollback

* Backup diario + previo a actualización.
* Restore con punto temporal identificado.
* Update con health checks post-despliegue.
* Rollback documentado a versión anterior con evidencia.

---

## 8. Flujos funcionales resumidos

### 8.1 Flujo de cobro

1. Caja abre cuenta en estado `abierta`.
2. Sistema valida que no haya ítems `en_preparacion`.
3. Se aplican descuentos válidos.
4. Se agrega propina.
5. Se registran pagos mixtos (si aplica).
6. Se calcula vuelto para efectivo.
7. Si total cubierto: cerrar cuenta, generar factura, imprimir/enviar.
8. Si no cubre: rechazar operación (no se permiten saldos pendientes).

### 8.2 Flujo de división de cuenta

1. Mesero/caja crea clientes de cuenta.
2. Asigna ítems a cliente.
3. Caja visualiza subtotales por cliente.
4. Puede cobrar por cliente o total consolidado en una sola transacción válida.

### 8.3 Flujo de cocina

* Cola única compartida en múltiples pantallas.
* Cada pantalla recibe mismos eventos base.
* El personal ejecuta transición de estados según producto que le corresponde.

---

## 9. Plan de pruebas obligatorio

Casos críticos mínimos:

1. Login seguro + bloqueo por intentos fallidos.
2. Aislamiento tenant en Nube.
3. Alta de mesa sin reordenar numeración existente.
4. Creación pedido y transición cocina.
5. Descuento válido/inválido por fecha y horario.
6. Cobro mixto con propina y vuelto.
7. Rechazo de cobro con pago incompleto.
8. División de cuenta por cliente.
9. Factura PDF e impresión.
10. Backup/restore/update/rollback.

---

## 10. Pendientes de decisión explícitos

1. **Reversión/anulación de cobro ya facturado:** pendiente de definición normativa y operativa.
2. **Agotados temporales por cocina/horario:** pendiente para versión futura.

---

**Fin del archivo `requirements.md`.**
