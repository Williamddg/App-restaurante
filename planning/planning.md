# Sprint Planning – Sistema de Gestión de Restaurante

**Basado en:** `requirements.md` (alineado y consolidado).  
**Metodología:** Scrum adaptado (sprints de 2 semanas; estimaciones ajustables).  
**Prioridad:** Seguridad temprana, base operativa robusta y entregables utilizables por restaurante real.

---

## Estructura general de sprints

| Sprint | Nombre | Objetivo central |
|---|---|---|
| 0 | Fundaciones seguras + arquitectura base | Seguridad de autenticación/sesión y base técnica estable |
| 1 | Núcleo operativo (mesas, cuentas, pedidos, cocina) | Flujo principal end-to-end sin cobro final |
| 2 | Caja completa (descuentos, propina, pagos mixtos, factura) | Cierre real de cuenta y comprobantes |
| 3 | División de cuenta por cliente + reportes base | Soporte operativo real en salón y analítica mínima |
| 4 | Tenant provisioning + despliegue Local/Nube + operación | Alta de restaurantes, health/startup checks, backups/restore |
| 5 | Inventario, hardening final, pruebas integrales y documentación | Cierre de calidad y salida a producción |

---

# Sprint 0: Fundaciones seguras + arquitectura base

## Objetivo
Levantar la plataforma base con autenticación robusta desde el inicio, sesiones endurecidas, auditoría de login y esqueleto multi-tenant compatible con app compartida + BD separada por tenant.

## Historias de usuario
- **HU-001:** Como usuario, quiero iniciar sesión de forma segura.
- **HU-002:** Como administrador, quiero que al desactivar usuarios se cierren sus sesiones.
- **HU-003:** Como proveedor, quiero una base de arquitectura compatible con tenants.

## Tareas técnicas

### 0.1 Infraestructura base
- [ ] Crear `compose.yaml` base (`app`, `db`, `proxy` opcional local).
- [ ] Definir `.env.example` con seguridad mínima obligatoria (`SESSION_SECRET`, cookies, timeouts).
- [ ] Implementar endpoints `/health/live` y `/health/ready`.
- [ ] Crear startup check inicial en `backend/src/bootstrap/startupChecks.ts`.

### 0.2 Backend de autenticación y sesiones
- [ ] Implementar `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`.
- [ ] Configurar cookies `HttpOnly`, `Secure`, `SameSite`.
- [ ] Implementar expiración por inactividad (30 min) y absoluta (8h).
- [ ] Crear tabla `user_sessions` y mecanismo de revocación.
- [ ] Invalidar sesiones al desactivar usuario.

### 0.3 Seguridad temprana obligatoria
- [ ] Rate limiting de login (`backend/src/middleware/rateLimit.ts`).
- [ ] Registrar intentos fallidos en `auth_attempts`.
- [ ] Validación de payloads de auth con esquema (`zod`/`joi`).
- [ ] Auditoría de eventos de login/logout.

### 0.4 Base de datos y modelos iniciales
- [ ] Modelos: `tenants`, `tenant_branding`, `tenant_settings`, `users`, `roles`, `permissions`, `user_roles`, `role_permissions`.
- [ ] Seed de roles base y admin inicial.
- [ ] Definir contrato de resolución de tenant (por subdominio/header/login).

## Criterios de aceptación Sprint 0
- [ ] Login funcional con sesión segura.
- [ ] Rate limiting activo en login.
- [ ] Sesiones expiran por inactividad y por límite absoluto.
- [ ] Al desactivar usuario se revocan sesiones activas.
- [ ] Health checks y startup checks funcionando.

---

# Sprint 1: Núcleo operativo (mesas, cuentas, pedidos, cocina)

## Objetivo
Implementar operación diaria principal: mesas, cuentas, pedidos y cocina en tiempo real.

## Historias de usuario
- **HU-004:** Como administrador, quiero agregar mesas sin reordenar las existentes.
- **HU-005:** Como mesero/kiosco, quiero crear pedidos y enviarlos a cocina.
- **HU-006:** Como cocina, quiero operar ítems por estado en una cola compartida.

## Tareas técnicas

### 1.1 Mesas y cuentas
- [ ] Modelo `tables` con `table_number` único por tenant y sin reordenamiento manual.
- [ ] API `POST /api/tables`, `GET /api/tables/status`, `PATCH /api/tables/{id}/disable`.
- [ ] Modelo `table_accounts` y apertura automática al primer pedido.
- [ ] Validar una sola cuenta activa por mesa.

### 1.2 Catálogo y medidas
- [ ] Modelos `menu_categories`, `dishes` con `measure_unit` (`unidad`, `g`, `kg`, `ml`, `l`).
- [ ] Registrar extras como platos independientes del catálogo.

### 1.3 Pedidos
- [ ] API `POST /api/orders` con `idempotency_key`.
- [ ] Estados de ítem: pendiente/en_preparacion/listo/entregado/cancelado.
- [ ] Cancelación sólo en `pendiente`.
- [ ] Snapshot de nombre/precio/medida en `order_items`.

### 1.4 Cocina (múltiples pantallas)
- [ ] API `GET /api/kitchen/queue`, `PATCH /api/kitchen/items/{id}/status`.
- [ ] Socket.IO para eventos de cola compartida.
- [ ] UI cocina mostrando mesa, producto, cantidad, medida, tiempo.
- [ ] Confirmar que varias pantallas ven la misma cola base.

## Criterios de aceptación Sprint 1
- [ ] Mesas nuevas se agregan sin alterar numeración previa.
- [ ] Kiosco/mesero crean pedidos correctos.
- [ ] Cocina recibe y actualiza ítems en tiempo real.
- [ ] Cola compartida visible en múltiples pantallas.

---

# Sprint 2: Caja completa (descuentos, propina, pagos mixtos, factura)

## Objetivo
Cerrar el circuito de cobro real con reglas completas de descuentos, propina, pagos mixtos y emisión de comprobantes.

## Historias de usuario
- **HU-007:** Como administrador, quiero configurar reglas de descuentos.
- **HU-008:** Como caja, quiero cobrar con múltiples métodos y calcular vuelto.
- **HU-009:** Como caja, quiero emitir, imprimir y reenviar factura PDF.

## Tareas técnicas

### 2.1 Descuentos
- [ ] Modelos `discount_rules` y `account_discounts`.
- [ ] API admin para CRUD de reglas (porcentaje/fijo, fechas, horarios, mínimo).
- [ ] Motor de validación/aplicación de descuentos en checkout.
- [ ] Persistir trazabilidad de regla aplicada.

### 2.2 Cobro
- [ ] API `POST /api/cash/accounts/{id}/checkout`.
- [ ] Soportar `tip_amount`.
- [ ] Soportar pagos mixtos en arreglo `payments[]`.
- [ ] Rechazar pago incompleto (sin saldo pendiente permitido).
- [ ] Calcular `change_amount` para efectivo.

### 2.3 Factura e impresión
- [ ] Secuencia de factura por tenant.
- [ ] Generación PDF con branding tenant.
- [ ] Impresión (ticket 80mm/A4 configurable).
- [ ] Reenvío por email con estado de envío.

### 2.4 Reportabilidad de caja
- [ ] Exponer datos de descuentos, propinas y métodos de pago para reportes.
- [ ] Auditoría de checkout, factura e impresión/reenvío.

## Criterios de aceptación Sprint 2
- [ ] Descuentos aplican correctamente por reglas.
- [ ] Cobro mixto funciona en una sola transacción.
- [ ] No se permite cerrar con saldo pendiente.
- [ ] Se calcula vuelto en efectivo.
- [ ] Factura PDF generada, imprimible y reenviable.

---

# Sprint 3: División de cuenta por cliente + reportes base

## Objetivo
Agregar división de cuenta por cliente y consolidar reportes operativos clave.

## Historias de usuario
- **HU-010:** Como mesero/caja, quiero dividir cuenta por cliente.
- **HU-011:** Como administrador, quiero filtrar reportes por cliente.

## Tareas técnicas

### 3.1 División de cuenta
- [ ] Modelos `account_customers` + referencia en `order_items.account_customer_id`.
- [ ] API `POST /api/accounts/{id}/customers`.
- [ ] API asignación de ítem a cliente.
- [ ] Vista de subtotales por cliente en caja.

### 3.2 Cobro por cliente o consolidado
- [ ] Permitir checkout por subcuenta (cliente) o total consolidado.
- [ ] Mantener regla de no saldo pendiente.

### 3.3 Reportes base
- [ ] `GET /api/reports/sales`.
- [ ] `GET /api/reports/discounts`.
- [ ] `GET /api/reports/tips`.
- [ ] `GET /api/reports/by-customer`.
- [ ] Exportación CSV/PDF.

## Criterios de aceptación Sprint 3
- [ ] Ítems se asignan a clientes correctamente.
- [ ] Caja puede cobrar por cliente o total.
- [ ] Reportes permiten filtro por cliente.

---

# Sprint 4: Tenant provisioning + despliegue Local/Nube + operación

## Objetivo
Aterrizar operación real de plataforma: aprovisionamiento de tenants, separación app compartida/BD dedicada, backups/restore/update/rollback y procedimientos operativos.

## Historias de usuario
- **HU-012:** Como proveedor, quiero provisionar un tenant con su BD y branding.
- **HU-013:** Como restaurante local, quiero operar con validación de licencia y ventana de gracia.
- **HU-014:** Como administrador, quiero backup/restore/update/rollback confiables.

## Tareas técnicas

### 4.1 Aprovisionamiento tenant
- [ ] Servicio `tenantProvisioningService`:
  - crear tenant,
  - crear BD dedicada,
  - ejecutar migraciones/seed,
  - cargar branding/settings,
  - crear usuario admin tenant.
- [ ] API interna `POST /api/platform/tenants`.

### 4.2 Resolución tenant en app compartida
- [ ] Middleware de resolución tenant.
- [ ] Conexión dinámica a BD dedicada por tenant.
- [ ] Pruebas de aislamiento entre tenants.

### 4.3 Local vs Nube
- [ ] Flujo Local: validación licencia + caché de última validación + gracia.
- [ ] Flujo Nube: validación suscripción tenant activa.
- [ ] Documentar comportamiento en pérdida de internet por modo.

### 4.4 Operación
- [ ] Scripts `backup.sh`, `restore.sh`, `update.sh`, `rollback.sh`.
- [ ] Backup diario + pre-update.
- [ ] Health checks post-update y rollback automático/manual.
- [ ] Runbooks operativos para soporte.

## Criterios de aceptación Sprint 4
- [ ] Tenant nuevo queda operativo con BD dedicada.
- [ ] App compartida funciona con múltiples tenants sin fuga de datos.
- [ ] Backups/restore funcionando y auditados.
- [ ] Update y rollback ejecutables con evidencia.

---

# Sprint 5: Inventario, hardening final, pruebas integrales y documentación

## Objetivo
Completar módulos restantes y cerrar calidad de release.

## Historias de usuario
- **HU-015:** Como administrador, quiero inventario manual con alertas.
- **HU-016:** Como equipo, quiero pruebas integrales y documentación lista para operación.

## Tareas técnicas

### 5.1 Inventario
- [ ] CRUD `inventory_items`, `inventory_lots`, `inventory_movements`.
- [ ] Alertas stock mínimo y vencimientos.
- [ ] Reporte inventario exportable.

### 5.2 Hardening final
- [ ] Revisar headers y políticas de seguridad.
- [ ] Revisar autorización endpoint por endpoint.
- [ ] Auditoría completa de acciones críticas restantes.

### 5.3 QA integral
- [ ] Unitarias + integración.
- [ ] E2E: flujo completo kiosco/mesero/cocina/caja/factura.
- [ ] Casos de descuentos, propina, pagos mixtos, cambio.
- [ ] Casos de división por cliente.
- [ ] Casos multi-tenant y operación (backup/restore/update/rollback).

### 5.4 Documentación final
- [ ] Manual Local y Nube.
- [ ] API docs (OpenAPI).
- [ ] Manual por rol.
- [ ] Runbook de incidentes.

## Criterios de aceptación Sprint 5
- [ ] Pruebas críticas aprobadas.
- [ ] Documentación completa y utilizable por operaciones.
- [ ] Release candidate apto para piloto.

---

## Backlog posterior (no obligatorio versión base)

1. Reversión/anulación de cobro facturado (requiere diseño normativo).
2. Gestión de agotados temporales por horario/cocina.
3. Partición avanzada automática de cola por estación de cocina.

---

## Resumen de dependencias críticas

1. Seguridad base (Sprint 0) bloquea salida a producción.
2. Núcleo operativo (Sprint 1) bloquea caja (Sprint 2).
3. Caja completa (Sprint 2) bloquea reportes de negocio sólidos (Sprint 3).
4. Tenant provisioning + operación (Sprint 4) bloquea despliegue SaaS estable.

---

**Fin del documento `planning.md`.**
