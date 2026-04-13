# Sprint Planning – Sistema de Gestión de Restaurante

**Basado en:** `requirements.md` (versión final con todas las modificaciones)  
**Metodología:** Scrum adaptado (sprints de 2 semanas, pero la duración es orientativa)  
**Enfoque:** Mini-tareas extremadamente detalladas, incluyendo nombres de archivos, rutas, funciones, métodos y criterios de aceptación.

---

## Estructura general de los sprints

| Sprint | Nombre | Duración estimada | Dependencias |
|--------|--------|------------------|--------------|
| 0 | Infraestructura base y autenticación | 2 semanas | Ninguna |
| 1 | Gestión de mesas y cuentas | 2 semanas | Sprint 0 |
| 2 | Módulo de pedidos (kiosco + mesero) y tiempo real | 2 semanas | Sprint 1 |
| 3 | Módulo de cocina y caja (facturación básica) | 2 semanas | Sprint 2 |
| 4 | Inventario, reportes y facturación avanzada | 2 semanas | Sprint 3 |
| 5 | Modo Nube, multi-tenant, backups y despliegue | 2 semanas | Sprint 4 |
| 6 | Seguridad, auditoría, pruebas y documentación final | 1 semana | Sprint 5 |

**Nota:** Cada sprint puede ajustarse en duración según el equipo. Las tareas están desglosadas al máximo nivel para que puedan ser asignadas a múltiples desarrolladores en paralelo.

---

# Sprint 0: Infraestructura base y autenticación

## Objetivo
Configurar el proyecto base (Docker, backend, frontend), crear la base de datos inicial e implementar el sistema de autenticación (login con email/contraseña, sesiones con cookies, roles básicos).

## Historias de usuario
- **HU-001:** Como administrador, quiero poder iniciar sesión con mi correo y contraseña para acceder al panel de administración.
- **HU-002:** Como mesero, quiero poder iniciar sesión para acceder a mis funciones.
- **HU-003:** Como sistema, quiero que las contraseñas estén hasheadas y las sesiones sean seguras.

## Tareas técnicas

### 0.1 Configuración del entorno Docker
- **Archivos:** `docker-compose.yml`, `Dockerfile.backend`, `Dockerfile.frontend`, `.env.example`
- **Sub-tareas:**
  - [ ] 0.1.1 Crear `docker-compose.yml` con servicios: `db` (MySQL 8), `backend` (Node.js), `frontend` (nginx sirviendo build).
  - [ ] 0.1.2 Configurar volúmenes para persistencia de MySQL (`mysql_data`) y para archivos estáticos (facturas, logs).
  - [ ] 0.1.3 Crear `Dockerfile.backend` basado en `node:18-alpine`, copiar `package.json`, instalar dependencias, copiar código fuente, exponer puerto 3000.
  - [ ] 0.1.4 Crear `Dockerfile.frontend` multi-etapa: build con Node, luego copiar a nginx:alpine.
  - [ ] 0.1.5 Crear `.env.example` con variables: `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `SESSION_SECRET`, `LICENSE_API_URL`, etc.
  - [ ] 0.1.6 Crear script `scripts/init-db.sql` para crear la base de datos y usuario inicial.
  - [ ] 0.1.7 Probar `docker compose up -d` y verificar que los contenedores arrancan.

### 0.2 Backend base (Node.js + Express + Prisma)
- **Archivos:** `backend/src/index.js`, `backend/src/app.js`, `backend/prisma/schema.prisma`, `backend/prisma/seed.js`
- **Sub-tareas:**
  - [ ] 0.2.1 Inicializar proyecto Node con `npm init`, instalar `express`, `prisma`, `@prisma/client`, `cors`, `helmet`, `express-session`, `bcrypt`, `dotenv`, `socket.io`.
  - [ ] 0.2.2 Configurar Prisma: definir modelo `User` con campos: `id`, `email` (único), `password_hash`, `full_name`, `role_id`, `restaurant_id` (opcional, luego se añade), `active`, `last_login_at`, `created_at`, `updated_at`.
  - [ ] 0.2.3 Definir modelos `Role` y `Permission` (para RBAC futuro). Inicialmente roles fijos: `admin`, `mesero`, `cocina`, `caja`, `kiosco`.
  - [ ] 0.2.4 Ejecutar `prisma migrate dev --name init` y generar cliente.
  - [ ] 0.2.5 Crear `seed.js` para insertar roles y un usuario administrador por defecto (email: admin@restaurant.com, password: admin123).
  - [ ] 0.2.6 Configurar `app.js` con middlewares: `express.json()`, `helmet()`, `cors()` (orígenes permitidos), `express-session` (store en memoria por ahora, luego Redis opcional).
  - [ ] 0.2.7 Crear ruta `POST /api/auth/login` que recibe `email`, `password`, valida con bcrypt, establece `req.session.userId` y `req.session.roleId`.
  - [ ] 0.2.8 Crear ruta `POST /api/auth/logout` que destruye sesión.
  - [ ] 0.2.9 Crear ruta `GET /api/auth/me` que devuelve usuario actual (sin hash).
  - [ ] 0.2.10 Crear middleware `isAuthenticated` y `hasPermission` (placeholder).
  - [ ] 0.2.11 Escribir pruebas unitarias con Vitest para login (mock de Prisma).

### 0.3 Frontend base (React + Vite + Tailwind)
- **Archivos:** `frontend/src/main.jsx`, `frontend/src/App.jsx`, `frontend/src/pages/Login.jsx`, `frontend/src/components/PrivateRoute.jsx`, `frontend/src/api/auth.js`
- **Sub-tareas:**
  - [ ] 0.3.1 Crear proyecto Vite con React y TypeScript (`npm create vite@latest frontend -- --template react-ts`).
  - [ ] 0.3.2 Instalar Tailwind CSS y configurar `tailwind.config.js`.
  - [ ] 0.3.3 Crear layout básico (sin autenticación): pantalla de login centrada.
  - [ ] 0.3.4 Implementar `Login.jsx` con formulario (email, password), llamada a `POST /api/auth/login`, guardar sesión (la cookie se maneja automáticamente).
  - [ ] 0.3.5 Implementar `PrivateRoute` que verifique si el usuario está autenticado (llamada a `/api/auth/me`); si no, redirige a login.
  - [ ] 0.3.6 Crear `api/client.js` con axios baseURL, interceptores para manejar errores de autenticación.
  - [ ] 0.3.7 Crear pantalla de dashboard genérica (por rol) que muestre "Bienvenido".
  - [ ] 0.3.8 Configurar rutas en `App.jsx`: `/login`, `/dashboard` (protegido), y redirección por defecto.
  - [ ] 0.3.9 Probar flujo de login/logout con el backend en Docker.

### 0.4 Base de datos de control de licencias (Supabase)
- **Archivos:** `license-service/` (proyecto separado o script en backend)
- **Sub-tareas:**
  - [ ] 0.4.1 Crear proyecto en Supabase (gratis).
  - [ ] 0.4.2 Crear tabla `licenses` con columnas: `license_key` (PK), `restaurant_code`, `status` (active/expired), `expires_at`, `allowed_channel`, `last_validated_at`.
  - [ ] 0.4.3 Crear tabla `subscriptions` para modo nube (similar).
  - [ ] 0.4.4 Crear API simple en el backend (o función serverless) para validar licencia: `POST /api/license/validate` que recibe `license_key` y devuelve estado.
  - [ ] 0.4.5 Integrar validación de licencia al iniciar el backend (solo en modo Local, pero lo dejamos para sprint 5).

### Criterios de aceptación del Sprint 0
- [ ] Ejecutando `docker compose up -d` se levantan todos los servicios.
- [ ] Se puede acceder a `http://localhost:3000` (backend) y `http://localhost` (frontend).
- [ ] Un usuario puede iniciar sesión con credenciales correctas y acceder al dashboard.
- [ ] Las contraseñas se almacenan hasheadas en la BD.
- [ ] La sesión expira después de 30 minutos de inactividad (configurable).
- [ ] El usuario administrador por defecto existe.

---

# Sprint 1: Gestión de mesas y cuentas

## Objetivo
Implementar el CRUD de mesas (solo administrador), el estado de mesas y la creación/administración de cuentas de mesa (abrir, cerrar, cancelar). También la gestión básica de usuarios y roles.

## Historias de usuario
- **HU-004:** Como administrador, quiero crear, editar y eliminar mesas (número fijo).
- **HU-005:** Como mesero, quiero ver el estado de todas las mesas (libre/ocupada/pagando).
- **HU-006:** Como sistema, al primer pedido de una mesa libre se debe crear automáticamente una cuenta.
- **HU-007:** Como administrador, quiero gestionar usuarios (crear, editar, desactivar) y asignar roles.

## Tareas técnicas

### 1.1 Modelos de datos (Prisma)
- **Archivo:** `backend/prisma/schema.prisma`
- **Sub-tareas:**
  - [ ] 1.1.1 Añadir modelo `Table` con: `id`, `restaurant_id` (para multi-tenant), `table_number` (único por restaurante), `enabled`, `display_order`.
  - [ ] 1.1.2 Añadir modelo `TableAccount` con: `id`, `table_id`, `status` (abierta, pagando, cerrada, cancelada), `opened_at`, `closed_at`, `total_amount`, `version` (control optimista).
  - [ ] 1.1.3 Añadir modelo `User` (ya existente) con relación a `restaurant_id` (opcional, luego).
  - [ ] 1.1.4 Ejecutar `prisma migrate dev --name add_tables_accounts`.
  - [ ] 1.1.5 Actualizar `seed.js` para crear mesas de ejemplo (10 mesas numeradas).

### 1.2 API REST para mesas
- **Archivos:** `backend/src/routes/tables.js`, `backend/src/controllers/tableController.js`
- **Sub-tareas:**
  - [ ] 1.2.1 Crear rutas CRUD protegidas por rol (solo admin): `GET /api/tables`, `POST /api/tables`, `PUT /api/tables/:id`, `DELETE /api/tables/:id`.
  - [ ] 1.2.2 En `POST /api/tables`, validar que `table_number` no exista.
  - [ ] 1.2.3 En `DELETE`, deshabilitar lógicamente (cambiar `enabled=false`) en lugar de borrar.
  - [ ] 1.2.4 Crear ruta `GET /api/tables/status` para obtener lista de mesas con su estado actual (libre/ocupada). Debe consultar si existe `TableAccount` activa para esa mesa.
  - [ ] 1.2.5 Escribir pruebas de integración con Vitest y supertest.

### 1.3 API para cuentas de mesa
- **Archivos:** `backend/src/routes/accounts.js`, `backend/src/controllers/accountController.js`
- **Sub-tareas:**
  - [ ] 1.3.1 Ruta `POST /api/accounts` (abrir nueva cuenta): requiere `table_id`. Verificar que la mesa no tenga cuenta abierta. Crear `TableAccount` con estado `abierta`.
  - [ ] 1.3.2 Ruta `GET /api/accounts/:id` para obtener detalle de una cuenta.
  - [ ] 1.3.3 Ruta `PUT /api/accounts/:id/close` para cerrar cuenta (solo si no hay platos en preparación, lógica posterior). Por ahora solo cambiar estado a `cerrada` si total = 0.
  - [ ] 1.3.4 Ruta `PUT /api/accounts/:id/cancel` para cancelar cuenta (solo si no tiene pedidos o todos pendientes).
  - [ ] 1.3.5 Implementar control de concurrencia con `version` (optimistic locking) para evitar actualizaciones simultáneas.

### 1.4 Gestión de usuarios y roles (administrador)
- **Archivos:** `backend/src/routes/users.js`, `backend/src/controllers/userController.js`, `frontend/src/pages/admin/Users.jsx`
- **Sub-tareas:**
  - [ ] 1.4.1 API `GET /api/users` (admin) listar todos los usuarios con paginación.
  - [ ] 1.4.2 API `POST /api/users` crear usuario (email, full_name, password, role_id). Hashear password.
  - [ ] 1.4.3 API `PUT /api/users/:id` editar (excepto password) y desactivar (campo `active`).
  - [ ] 1.4.4 API `DELETE /api/users/:id` (desactivar, no borrar físicamente).
  - [ ] 1.4.5 API `GET /api/roles` para obtener lista de roles y permisos (fijos).
  - [ ] 1.4.6 En el frontend, crear vista `Users.jsx` con tabla, formulario modal para crear/editar, y botón de desactivar.
  - [ ] 1.4.7 Proteger todas las rutas con middleware `hasPermission('users.manage')`.

### 1.5 Frontend: vista de mesas para mesero y caja
- **Archivos:** `frontend/src/pages/waitress/Tables.jsx`, `frontend/src/components/TableCard.jsx`, `frontend/src/api/tables.js`
- **Sub-tareas:**
  - [ ] 1.5.1 Crear componente `TableCard` que muestre número de mesa, estado (color: verde libre, rojo ocupada, amarillo pagando) y un botón "Ver cuenta" si está ocupada.
  - [ ] 1.5.2 Llamar a `GET /api/tables/status` periódicamente (cada 10 segundos o con WebSockets futuro) para actualizar estados.
  - [ ] 1.5.3 Al hacer clic en "Ver cuenta", redirigir a `/table/:id` (detalle de cuenta, se implementa en sprint 2).
  - [ ] 1.5.4 Para administrador, añadir botón "Administrar mesas" que abra un modal para editar números y habilitar/deshabilitar.

### Criterios de aceptación del Sprint 1
- [ ] El administrador puede crear mesas numeradas y verlas en la lista.
- [ ] El mesero ve las mesas con colores según estado.
- [ ] Al crear una cuenta (vía API), la mesa cambia a estado `ocupada`.
- [ ] El administrador puede crear usuarios y asignar roles.
- [ ] Los usuarios no administradores no pueden acceder a rutas de administración.

---

# Sprint 2: Módulo de pedidos (kiosco + mesero) y tiempo real

## Objetivo
Implementar la creación de pedidos desde kiosco y desde mesero, con selección de platos, observaciones, y envío a cocina. Integrar WebSockets (Socket.IO) para notificar nuevos pedidos en tiempo real.

## Historias de usuario
- **HU-008:** Como cliente en kiosco, quiero iniciar sesión, seleccionar una mesa libre, elegir platos del menú y enviar el pedido.
- **HU-009:** Como mesero, quiero agregar platos a una mesa ya ocupada, con observaciones.
- **HU-010:** Como sistema, al enviar un pedido, debe crearse un ticket con platos y notificar a cocina.
- **HU-011:** Como mesero, quiero cancelar un plato que aún esté pendiente.

## Tareas técnicas

### 2.1 Modelos de datos adicionales
- **Archivo:** `backend/prisma/schema.prisma`
- **Sub-tareas:**
  - [ ] 2.1.1 Añadir modelo `MenuCategory` y `Dish` (según el modelo de datos en requirements).
  - [ ] 2.1.2 Añadir modelo `OrderTicket` y `OrderItem`.
  - [ ] 2.1.3 Añadir relaciones: `TableAccount` tiene muchos `OrderTicket`, cada `OrderTicket` tiene muchos `OrderItem`.
  - [ ] 2.1.4 Añadir campo `origin` en `OrderTicket` (`kiosco`, `mesero`, `caja`).
  - [ ] 2.1.5 Ejecutar migración y seed de platos de ejemplo (pizzas, bebidas, etc.).

### 2.2 API de catálogo (platos)
- **Archivos:** `backend/src/routes/dishes.js`, `backend/src/controllers/dishController.js`
- **Sub-tareas:**
  - [ ] 2.2.1 `GET /api/dishes` listar platos activos con categorías, precios, imágenes (url placeholder).
  - [ ] 2.2.2 `GET /api/dishes/categories` listar categorías.
  - [ ] 2.2.3 Rutas protegidas para admin: `POST`, `PUT`, `DELETE` para platos y categorías.

### 2.3 API de pedidos
- **Archivos:** `backend/src/routes/orders.js`, `backend/src/controllers/orderController.js`
- **Sub-tareas:**
  - [ ] 2.3.1 `POST /api/orders` – crear pedido (ticket + items). Requiere `table_account_id`, `origin`, array de `{dish_id, quantity, notes}`.
  - [ ] 2.3.2 Validar que la cuenta esté `abierta`. Si no existe, crearla automáticamente (llamada interna).
  - [ ] 2.3.3 Para cada item, guardar snapshot de nombre y precio del plato actual.
  - [ ] 2.3.4 Calcular subtotal y actualizar `total_amount` en `TableAccount` (sumar).
  - [ ] 2.3.5 Emitir evento Socket.IO `order_item.created` al canal `kitchen` y `cash` con los nuevos items.
  - [ ] 2.3.6 `DELETE /api/orders/item/:itemId` – cancelar un item solo si estado = `pendiente`. Cambiar estado a `cancelado`. Emitir evento de actualización.
  - [ ] 2.3.7 Implementar idempotencia con un token único en la petición para evitar duplicados.

### 2.4 WebSockets con Socket.IO
- **Archivos:** `backend/src/socket.js`, `frontend/src/socket.js`
- **Sub-tareas:**
  - [ ] 2.4.1 En backend, crear servidor Socket.IO adjunto al servidor HTTP.
  - [ ] 2.4.2 Middleware de autenticación: obtener `sessionId` desde la cookie, validar usuario.
  - [ ] 2.4.3 Unir a salas: `tenant:{restaurantId}`, `user:{userId}`.
  - [ ] 2.4.4 Emitir eventos desde controladores de pedidos.
  - [ ] 2.4.5 En frontend, crear hook `useSocket` que se conecte y escuche eventos según el rol (cocina escucha `order_item.created`, mesero escucha `order_item.updated`).
  - [ ] 2.4.6 Actualizar la interfaz de mesas en tiempo real cuando un pedido es creado o cancelado.

### 2.5 Frontend – Kiosco
- **Archivos:** `frontend/src/pages/kiosk/KioskLogin.jsx`, `frontend/src/pages/kiosk/MesaSelection.jsx`, `frontend/src/pages/kiosk/Menu.jsx`, `frontend/src/pages/kiosk/OrderSummary.jsx`
- **Sub-tareas:**
  - [ ] 2.5.1 Pantalla de login para kiosco (usuario/contraseña). Guarda sesión.
  - [ ] 2.5.2 Tras login, mostrar selector de mesas libres (llamada a `GET /api/tables/status` filtrando libres).
  - [ ] 2.5.3 Al seleccionar mesa, redirigir a menú (`/kiosk/menu?tableId=...`).
  - [ ] 2.5.4 Componente `Menu` que muestra categorías, platos con imagen, precio, botón "Agregar". Usa carrito local (en memoria).
  - [ ] 2.5.5 Pantalla `OrderSummary` con lista de items, cantidades, total. Botón "Enviar pedido".
  - [ ] 2.5.6 Al enviar, llama a `POST /api/orders` y luego limpia carrito y redirige a inicio del kiosco (mesa liberada).
  - [ ] 2.5.7 Diseño táctil con botones grandes, sin opciones de cancelación.

### 2.6 Frontend – Mesero (tablet)
- **Archivos:** `frontend/src/pages/waitress/TableDetail.jsx`, `frontend/src/components/OrderItemList.jsx`, `frontend/src/components/DishSelector.jsx`
- **Sub-tareas:**
  - [ ] 2.6.1 Vista `TableDetail` que muestra la cuenta actual de una mesa (items, subtotal, total).
  - [ ] 2.6.2 Botón "Agregar plato" que abre modal `DishSelector` con búsqueda y selección de cantidad/observaciones.
  - [ ] 2.6.3 Al confirmar, llama a `POST /api/orders`.
  - [ ] 2.6.4 Cada item tiene botón "Cancelar" si estado es `pendiente`. Al cancelar, llama a `DELETE /api/orders/item/:itemId`.
  - [ ] 2.6.5 Escuchar eventos Socket.IO para actualizar la lista de items en tiempo real.

### Criterios de aceptación del Sprint 2
- [ ] El kiosco permite login, seleccionar mesa, agregar platos al carrito y enviar pedido.
- [ ] El pedido aparece en la base de datos con los items correctos.
- [ ] El mesero puede agregar platos a una mesa ocupada.
- [ ] El mesero puede cancelar platos pendientes.
- [ ] Los cambios se reflejan en tiempo real en la interfaz de mesero (sin recargar).
- [ ] La cocina recibe el evento (aunque aún no tiene interfaz, se puede ver por logs).

---

# Sprint 3: Módulo de cocina y caja (facturación básica)

## Objetivo
Implementar la interfaz de cocina (cambiar estados de platos), y la interfaz de caja (cerrar cuentas, generar factura PDF básica, registrar pagos). También la impresión física de facturas.

## Historias de usuario
- **HU-012:** Como cocina, quiero ver la cola de platos pendientes, marcar "en preparación" y "listo".
- **HU-013:** Como caja, quiero ver las cuentas activas, cerrar una cuenta, registrar el método de pago y generar factura.
- **HU-014:** Como caja, quiero imprimir la factura en una impresora térmica.
- **HU-015:** Como caja, quiero enviar la factura por email al cliente.

## Tareas técnicas

### 3.1 API de cocina
- **Archivos:** `backend/src/routes/kitchen.js`, `backend/src/controllers/kitchenController.js`
- **Sub-tareas:**
  - [ ] 3.1.1 `GET /api/kitchen/items` – obtener items con estado `pendiente` o `en_preparacion`, ordenados por antigüedad.
  - [ ] 3.1.2 `PUT /api/kitchen/items/:itemId/status` – cambiar estado a `en_preparacion` o `listo`. Validar transiciones permitidas.
  - [ ] 3.1.3 Emitir eventos Socket.IO `item.updated` para que mesero y caja vean cambios.
  - [ ] 3.1.4 Al marcar `listo`, actualizar también el estado del plato en la cuenta.

### 3.2 Frontend – Cocina
- **Archivos:** `frontend/src/pages/kitchen/KitchenQueue.jsx`, `frontend/src/components/KitchenItemCard.jsx`
- **Sub-tareas:**
  - [ ] 3.2.1 Mostrar dos columnas: "Pendientes" y "En preparación". Cada item muestra número de mesa, nombre del plato, observaciones, tiempo transcurrido.
  - [ ] 3.2.2 Botones "Iniciar" (mover a en preparación) y "Listo".
  - [ ] 3.2.3 Escuchar eventos Socket.IO para actualizar la cola automáticamente.
  - [ ] 3.2.4 Sonido opcional al recibir nuevo pedido.

### 3.3 API de caja y facturación
- **Archivos:** `backend/src/routes/cash.js`, `backend/src/controllers/cashController.js`, `backend/src/services/invoiceGenerator.js`
- **Sub-tareas:**
  - [ ] 3.3.1 `GET /api/cash/accounts` – listar cuentas abiertas con datos de mesa y total.
  - [ ] 3.3.2 `POST /api/cash/accounts/:id/pay` – registrar pago (método, monto). Si el monto cubre el total, cambiar estado de cuenta a `pagando` y luego a `cerrada` (tras factura).
  - [ ] 3.3.3 Generar factura PDF con Puppeteer: plantilla HTML con datos del restaurante (logo, nombre, etc.) y detalles de la cuenta.
  - [ ] 3.3.4 Guardar PDF en `facturas/año/mes/dia/factura_{numero}.pdf` (crear directorios automáticamente).
  - [ ] 3.3.5 Insertar registro en tabla `invoices` con número consecutivo (usar secuencia por restaurante).
  - [ ] 3.3.6 Enviar email con Nodemailer (configuración SMTP desde variables de entorno). Adjuntar PDF.
  - [ ] 3.3.7 Ruta `POST /api/invoices/:id/resend` para reenviar.
  - [ ] 3.3.8 Ruta `GET /api/invoices/:id/pdf` para descargar PDF.
  - [ ] 3.3.9 Implementar impresión física: endpoint `POST /api/invoices/:id/print` que devuelve HTML/CSS para impresora térmica (80mm). El frontend abrirá diálogo de impresión.

### 3.4 Frontend – Caja
- **Archivos:** `frontend/src/pages/cash/CashDashboard.jsx`, `frontend/src/pages/cash/AccountDetail.jsx`, `frontend/src/components/PrintInvoice.jsx`
- **Sub-tareas:**
  - [ ] 3.4.1 Vista `CashDashboard` con lista de mesas con cuentas abiertas, mostrando número de mesa y total.
  - [ ] 3.4.2 Al seleccionar una cuenta, ir a `AccountDetail`.
  - [ ] 3.4.3 En `AccountDetail` mostrar items, subtotal, total. Botón "Cobrar".
  - [ ] 3.4.4 Modal de cobro: seleccionar método de pago (efectivo, tarjeta, etc.), ingresar monto, email del cliente (opcional para factura).
  - [ ] 3.4.5 Al confirmar, llamar a `POST /api/cash/accounts/:id/pay`. Si éxito, mostrar factura PDF y opciones: "Enviar por email", "Imprimir", "Descargar".
  - [ ] 3.4.6 Función de impresión: abrir ventana con HTML generado por el backend y llamar a `window.print()`.
  - [ ] 3.4.7 Si la opción `cash_can_create_orders` está activa, mostrar botón "Agregar pedido" en la cuenta (reutilizar componente de mesero).

### Criterios de aceptación del Sprint 3
- [ ] La cocina puede cambiar estados y la cola se actualiza en tiempo real.
- [ ] La caja puede cerrar una cuenta, generar factura PDF y enviarla por email.
- [ ] La factura se guarda en la estructura de carpetas.
- [ ] Se puede imprimir la factura desde el navegador.
- [ ] Los métodos de pago son configurables por administrador.

---

# Sprint 4: Inventario, reportes y facturación avanzada

## Objetivo
Implementar el módulo de inventario (solo administrador) con alertas de stock mínimo, gestión de proveedores y lotes. Generar reportes avanzados (ventas, platos más vendidos, etc.) con exportación PDF/CSV. Mejorar la facturación con numeración consecutiva.

## Historias de usuario
- **HU-016:** Como administrador, quiero gestionar el inventario (productos, lotes, proveedores) y recibir alertas de stock bajo.
- **HU-017:** Como administrador, quiero ver reportes de ventas, desempeño de meseros, platos más vendidos, etc.
- **HU-018:** Como administrador, quiero exportar reportes a PDF y CSV.

## Tareas técnicas

### 4.1 Modelos de inventario
- **Archivos:** `backend/prisma/schema.prisma` (ya definidos en requirements, asegurarse de que estén)
- **Sub-tareas:**
  - [ ] 4.1.1 Verificar existencia de modelos `InventoryItem`, `Supplier`, `InventoryLot`, `InventoryMovement`.
  - [ ] 4.1.2 Ejecutar migración si faltan.
  - [ ] 4.1.3 Crear seeds de productos de ejemplo.

### 4.2 API de inventario
- **Archivos:** `backend/src/routes/inventory.js`, `backend/src/controllers/inventoryController.js`
- **Sub-tareas:**
  - [ ] 4.2.1 CRUD de productos (solo admin): `GET /api/inventory/items`, `POST`, `PUT`, `DELETE`.
  - [ ] 4.2.2 CRUD de proveedores.
  - [ ] 4.2.3 Gestión de lotes: `POST /api/inventory/lots` (recibir lote con cantidad, fecha vencimiento).
  - [ ] 4.2.4 Registrar movimientos: `POST /api/inventory/movements` (tipo: ingreso, consumo, desperdicio, ajuste). Actualizar stock.
  - [ ] 4.2.5 Endpoint `GET /api/inventory/alerts` para obtener productos con stock < mínimo y productos próximos a vencer (7 días).
  - [ ] 4.2.6 Programar una tarea (cron job dentro del backend) que revise alertas diariamente y envíe email al administrador.

### 4.3 API de reportes
- **Archivos:** `backend/src/routes/reports.js`, `backend/src/controllers/reportController.js`, `backend/src/services/reportGenerator.js`
- **Sub-tareas:**
  - [ ] 4.3.1 `GET /api/reports/sales` – ventas totales por rango de fechas, agrupado por día, método de pago, origen.
  - [ ] 4.3.2 `GET /api/reports/top-dishes` – platos más vendidos (por cantidad).
  - [ ] 4.3.3 `GET /api/reports/waiters-performance` – ventas por mesero, número de pedidos.
  - [ ] 4.3.4 `GET /api/reports/kitchen-performance` – tiempo promedio de preparación.
  - [ ] 4.3.5 `GET /api/reports/inventory` – movimientos y stock actual.
  - [ ] 4.3.6 Para cada reporte, aceptar parámetros `format=pdf` o `csv`. Usar librería `pdfkit` o `puppeteer` para PDF, y `json2csv` para CSV.
  - [ ] 4.3.7 Guardar reportes generados? No es necesario, se generan bajo demanda.

### 4.4 Frontend – Inventario y reportes
- **Archivos:** `frontend/src/pages/admin/Inventory.jsx`, `frontend/src/pages/admin/Reports.jsx`, `frontend/src/components/ReportFilters.jsx`
- **Sub-tareas:**
  - [ ] 4.4.1 Vista `Inventory` con lista de productos, stock actual, mínimo. Botón para agregar/editar, gestionar lotes, registrar movimientos.
  - [ ] 4.4.2 Mostrar alertas en un panel superior.
  - [ ] 4.4.3 Vista `Reports` con selector de tipo de reporte, rango de fechas, filtros adicionales (mesero, método pago). Botón "Generar" y luego "Exportar PDF" / "Exportar CSV".
  - [ ] 4.4.4 Al generar, llamar a la API correspondiente con los filtros y el formato, luego descargar el archivo.

### Criterios de aceptación del Sprint 4
- [ ] El administrador puede agregar productos de inventario y gestionar lotes.
- [ ] Las alertas de stock mínimo y vencimiento se muestran correctamente.
- [ ] Los reportes se generan con los filtros y se exportan a PDF/CSV.
- [ ] Los reportes contienen datos reales basados en pedidos y pagos.

---

# Sprint 5: Modo Nube, multi-tenant, backups y despliegue

## Objetivo
Implementar el soporte para modo Nube (multi-tenant con base de datos por restaurante) y modo Local (validación de licencia). Añadir backups automáticos (locales y a la nube) y scripts de despliegue/actualización con Docker.

## Historias de usuario
- **HU-019:** Como administrador del proveedor, quiero poder aprovisionar un nuevo restaurante en la nube con su propia base de datos y branding.
- **HU-020:** Como dueño de restaurante en modo Local, quiero que el sistema valide mi licencia al iniciar y funcione sin internet durante un período de gracia.
- **HU-021:** Como administrador, quiero programar backups automáticos y poder restaurar desde un backup.
- **HU-022:** Como administrador, quiero actualizar el sistema a una nueva versión y poder hacer rollback si falla.

## Tareas técnicas

### 5.1 Multi-tenant en modo Nube
- **Archivos:** `backend/src/middleware/tenant.js`, `backend/prisma/` (múltiples conexiones), `backend/src/services/tenantManager.js`
- **Sub-tareas:**
  - [ ] 5.1.1 Modificar esquema: añadir `restaurant_id` a todas las tablas operativas. O usar base de datos separada por restaurante (recomendado). Elegir la segunda opción: cada restaurante tiene su propia base de datos MySQL.
  - [ ] 5.1.2 Crear un servicio de gestión de tenants: al crear un nuevo restaurante, ejecutar script que cree una nueva base de datos (con nombre `rest_{code}`) y ejecute las migraciones.
  - [ ] 5.1.3 En el backend, usar un pool de conexiones por tenant (o crear conexión dinámica según subdominio o header `X-Tenant-ID`).
  - [ ] 5.1.4 Implementar middleware que identifique el tenant a partir del dominio (ej. `restaurante1.sistema.com`) o de un campo en el login.
  - [ ] 5.1.5 Ajustar todas las consultas para que usen el cliente Prisma correspondiente al tenant.
  - [ ] 5.1.6 Crear API para el panel de control del proveedor: `POST /api/admin/tenants` (crear nuevo tenant), `GET /api/admin/tenants`, etc.

### 5.2 Validación de licencias (modo Local)
- **Archivos:** `backend/src/services/license.js`, `backend/src/cron/licenseCheck.js`
- **Sub-tareas:**
  - [ ] 5.2.1 Al iniciar el backend en modo Local, leer `LICENSE_KEY` desde variables de entorno.
  - [ ] 5.2.2 Llamar a la API central (Supabase) para validar la licencia. Guardar en caché local (archivo o variable) el estado y la fecha de expiración.
  - [ ] 5.2.3 Programar tarea cada 24 horas para revalidar. Si no hay internet, usar la caché (período de gracia de 7 días).
  - [ ] 5.2.4 Si la licencia no es válida o expiró, el backend debe rechazar todas las peticiones excepto una ruta de administración que muestre mensaje de contacto.
  - [ ] 5.2.5 En modo Nube, la validación se basa en la suscripción activa del tenant (consultar base de control).

### 5.3 Backups automáticos
- **Archivos:** `backend/src/scripts/backup.js`, `backend/src/cron/backupScheduler.js`
- **Sub-tareas:**
  - [ ] 5.3.1 Crear script que ejecute `mysqldump` de la base de datos (o de todas las bases en modo Nube) y comprima el archivo.
  - [ ] 5.3.2 Guardar backup localmente en `./backups/` con nombre `backup_YYYYMMDD_HHMMSS.sql.gz`.
  - [ ] 5.3.3 Usar `rclone` para subir a la nube configurada (Google Drive, S3, etc.). El administrador puede configurar el proveedor desde el panel.
  - [ ] 5.3.4 Programar tarea cron (dentro del contenedor) para ejecutar backup diario a las 3 AM.
  - [ ] 5.3.5 Proveer API `POST /api/admin/backup` para ejecutar manual y `GET /api/admin/backups` para listar backups disponibles.
  - [ ] 5.3.6 Implementar restauración: `POST /api/admin/restore` que recibe un archivo backup y lo restaura (solo accesible para admin).

### 5.4 Actualización y rollback con Docker
- **Archivos:** `scripts/update.sh`, `scripts/rollback.sh`, `docker-compose.override.yml`
- **Sub-tareas:**
  - [ ] 5.4.1 Crear `update.sh` que:
    - Hace backup de BD.
    - Descarga nuevas imágenes (docker compose pull).
    - Recrea contenedores con `docker compose up -d`.
    - Ejecuta health-check (endpoint `/health`).
    - Si falla, ejecuta `rollback.sh`.
  - [ ] 5.4.2 `rollback.sh` que vuelve a la imagen anterior (usando etiqueta `previous` o guardando el tag anterior en un archivo).
  - [ ] 5.4.3 Añadir endpoint `/api/admin/version` que devuelve la versión actual y la última disponible (consultando un archivo JSON en GitHub Releases).
  - [ ] 5.4.4 En el panel de administración, añadir sección "Actualizaciones" con botón "Buscar actualizaciones" y "Actualizar ahora".

### 5.5 Configuración de red y puertos (modo Local)
- **Archivos:** `backend/src/routes/admin/settings.js`, `frontend/src/pages/admin/NetworkSettings.jsx`
- **Sub-tareas:**
  - [ ] 5.5.1 API para cambiar puerto HTTP, puerto MySQL, IP de bind (requiere reautenticación).
  - [ ] 5.5.2 Almacenar configuración en un archivo `.env` o en variables de entorno.
  - [ ] 5.5.3 Interfaz para que el administrador modifique estos valores y reinicie servicios (o al menos muestre instrucciones).

### Criterios de aceptación del Sprint 5
- [ ] En modo Nube, se pueden crear múltiples restaurantes con bases de datos separadas y branding independiente.
- [ ] En modo Local, el sistema valida licencia al inicio y funciona sin internet durante el período de gracia.
- [ ] Los backups automáticos se generan diariamente y se suben a la nube.
- [ ] Se puede restaurar desde un backup.
- [ ] El administrador puede actualizar el sistema a una nueva versión y hacer rollback.

---

# Sprint 6: Seguridad, auditoría, pruebas y documentación final

## Objetivo
Reforzar la seguridad (RBAC completo, auditoría, protección contra inyección), realizar pruebas exhaustivas (unitarias, integración, E2E, carga) y completar la documentación técnica y de usuario.

## Historias de usuario
- **HU-023:** Como administrador, quiero ver un registro de auditoría de todas las acciones importantes.
- **HU-024:** Como administrador, quiero asignar permisos específicos a roles (por ejemplo, que un mesero no pueda cerrar cuentas).
- **HU-025:** Como equipo, quiero tener pruebas automatizadas para evitar regresiones.

## Tareas técnicas

### 6.1 Auditoría completa
- **Archivos:** `backend/src/middleware/audit.js`, `backend/prisma/schema.prisma` (tabla `AuditLog` ya existe)
- **Sub-tareas:**
  - [ ] 6.1.1 Crear middleware que capture antes/después de mutaciones críticas y registre en `AuditLog`.
  - [ ] 6.1.2 En cada controlador importante, llamar a `auditService.log(userId, action, entityType, entityId, before, after, ip, userAgent)`.
  - [ ] 6.1.3 API `GET /api/admin/audit` con filtros (fecha, usuario, acción) y paginación.
  - [ ] 6.1.4 Vista en frontend para que el administrador consulte la auditoría.

### 6.2 RBAC granular y permisos
- **Archivos:** `backend/src/middleware/rbac.js`, `frontend/src/utils/permissions.js`
- **Sub-tareas:**
  - [ ] 6.2.1 Terminar la tabla `permissions` y `role_permissions`. Insertar todos los permisos listados en requirements.
  - [ ] 6.2.2 Crear roles predefinidos con sus permisos.
  - [ ] 6.2.3 Implementar middleware `hasPermission(permission)` que verifique el rol del usuario.
  - [ ] 6.2.4 En frontend, crear hook `usePermissions` para mostrar/ocultar elementos de UI según permisos.
  - [ ] 6.2.5 Panel de administración para gestionar roles y permisos (asignar permisos a roles).

### 6.3 Pruebas de seguridad
- **Sub-tareas:**
  - [ ] 6.3.1 Probar inyección SQL en todos los endpoints (usar payloads maliciosos) – Prisma ya protege, pero verificar.
  - [ ] 6.3.2 Probar XSS en campos de texto (observaciones, nombres).
  - [ ] 6.3.3 Probar que las rutas protegidas no son accesibles sin autenticación.
  - [ ] 6.3.4 Probar que un usuario no puede acceder a datos de otro tenant (en modo Nube).
  - [ ] 6.3.5 Probar rate limiting en login.

### 6.4 Pruebas de carga (k6)
- **Archivos:** `tests/load/k6-script.js`
- **Sub-tareas:**
  - [ ] 6.4.1 Simular 20 usuarios concurrentes realizando pedidos, consultas de estado, etc.
  - [ ] 6.4.2 Verificar que los tiempos de respuesta están dentro de lo aceptable (ver sección 4.1).

### 6.5 Pruebas end-to-end (Playwright)
- **Archivos:** `tests/e2e/`
- **Sub-tareas:**
  - [ ] 6.5.1 Flujo completo: kiosco hace pedido, cocina lo prepara, caja cobra y genera factura.
  - [ ] 6.5.2 Flujo de mesero cancelando plato.
  - [ ] 6.5.3 Flujo de administración: crear usuario, mesa, plato.
  - [ ] 6.5.4 Ejecutar en CI (GitHub Actions) con Docker.

### 6.6 Documentación
- **Archivos:** `docs/` (manuales de usuario, despliegue, API)
- **Sub-tareas:**
  - [ ] 6.6.1 Manual de despliegue para modo Local (paso a paso con Docker).
  - [ ] 6.6.2 Manual de despliegue para modo Nube (aprovisionamiento de tenant).
  - [ ] 6.6.3 Manual de usuario para kiosco, mesero, cocina, caja, administrador.
  - [ ] 6.6.4 Documentación de API (Swagger/OpenAPI).
  - [ ] 6.6.5 Runbooks de actualización, rollback, backup y restauración.
  - [ ] 6.6.6 README principal con instrucciones de desarrollo.

### Criterios de aceptación del Sprint 6
- [ ] Todos los endpoints tienen autorización basada en permisos.
- [ ] La auditoría registra todas las acciones críticas.
- [ ] Las pruebas de seguridad no encuentran vulnerabilidades graves.
- [ ] Las pruebas E2E pasan sin errores.
- [ ] La documentación está completa y clara.

---

## Resumen de entregables por sprint

| Sprint | Entregables principales |
|--------|-------------------------|
| 0 | Docker compose, backend con autenticación, frontend con login, base de datos inicial. |
| 1 | CRUD de mesas, gestión de cuentas, administración de usuarios. |
| 2 | Catálogo de platos, pedidos desde kiosco y mesero, WebSockets. |
| 3 | Interfaz de cocina, caja, factura PDF, impresión y email. |
| 4 | Inventario, alertas, reportes avanzados, exportación. |
| 5 | Multi-tenant, licencias, backups automáticos, actualizaciones con Docker. |
| 6 | Seguridad, auditoría, pruebas, documentación final. |

---

**Fin del documento `planning.md`**