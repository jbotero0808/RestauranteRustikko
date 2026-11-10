# 🍽️ Gestión Restaurante

Aplicación web para la gestión integral de un restaurante: menú digital, pedidos en tiempo real con pantalla de cocina (KDS), reservas, cierre de caja y reportes de ventas.

> **Versión actual:** 0.8.0 — Vista del salón y cuenta por mesa. Ver el [CHANGELOG](CHANGELOG.md).

## Estado de los módulos

| Módulo | Estado |
|--------|--------|
| Autenticación, usuarios y roles | ✅ Disponible |
| Configuración del restaurante (datos, logo, horarios, moneda, impuestos) | ✅ Disponible |
| Menú digital (categorías y platos) | ✅ Disponible |
| Mesas: vista del salón en tiempo real, cuenta por mesa con adiciones y cobro de la mesa completa (se configuran en Configuración) | ✅ Disponible |
| Pedidos en mesa, para llevar y a **domicilio**, en tiempo real, con pantalla de cocina (KDS) y cobro (efectivo o transferencia) | ✅ Disponible |
| Caja: apertura, salidas de efectivo con categoría, pago automático a domiciliarios, cierre y arqueo | ✅ Disponible |
| Gastos (nómina, arriendo, servicios…) con categorías configurables | ✅ Disponible |
| Reportes: ventas diarias y mensuales, ingresos − gastos = ganancia, panel de inicio | ✅ Disponible |
| Reservas | 🚧 Paso 5 |

### Flujo de un pedido

1. El **administrador** abre la caja en **Caja** (necesario para poder cobrar).
2. El **mesero** toma el pedido desde **Mesas** (tocando una mesa libre) o en **Pedidos → + Nuevo pedido**. El cliente también puede pedir desde el menú.
3. **Cocina** lo ve aparecer al instante en **Cocina (KDS)** y lo pasa a *En preparación* y luego a *Listo*.
4. Si la mesa pide algo más, en **Mesas → (mesa) → ➕ Adicionar**: se suma a la misma cuenta y la cocina lo ve como «ADICIÓN».
5. Para cobrar: **Mesas → (mesa) → 💰 Cobrar mesa** cobra toda la cuenta (pedido inicial + adiciones) y deja la mesa libre. Para llevar y domicilios se cobran desde **Pedidos → Cobrar**. Solo efectivo o transferencia; no se cobra propina.
6. **Domicilios:** al cobrarlos, el pago al domiciliario se registra solo como salida de caja (categoría «Domicilios», con el nombre del cliente), aunque el cliente haya pagado por transferencia.
7. Si se saca efectivo durante el día (p. ej. una compra en la tienda), el **administrador** lo registra en **Caja → Salidas de efectivo** eligiendo una categoría.
8. Al final del día el **administrador** cierra la caja con el arqueo: efectivo esperado = inicial + ventas en efectivo − salidas.
9. Los gastos que no salen de la caja (arriendo, servicios, nómina…) se registran en **Gastos**, indicando el mes al que corresponden.
10. En **Reportes** se ve, por mes: ingresos − gastos = **ganancia**, con el detalle por categoría.

## Tecnologías

| Capa | Tecnología |
|------|-----------|
| Frontend | React 18, Vite 6, Tailwind CSS 3, React Router 6, Axios |
| Backend | Node.js, Express 5, Zod, Socket.io |
| Base de datos | PostgreSQL 14+ con Prisma ORM 6 |
| Seguridad | JWT, bcrypt, Helmet, rate limiting, control de acceso por roles (RBAC) |

## Requisitos previos

| Herramienta | Versión | Verificar con |
|-------------|---------|---------------|
| [Node.js](https://nodejs.org/) | **18.18 o superior** | `node -v` |
| npm | 9 o superior | `npm -v` |
| [PostgreSQL](https://www.postgresql.org/download/) | 14 o superior | `psql --version` |
| Git | cualquiera | `git --version` |

## Instalación y ejecución

### 1. Clonar el repositorio

```bash
git clone <URL_DEL_REPOSITORIO> Gestion_Restaurante
cd Gestion_Restaurante
```

### 2. Crear la base de datos

Con un usuario administrador de PostgreSQL (en Linux normalmente `postgres`):

```bash
sudo -u postgres psql
```

```sql
CREATE USER restaurante WITH PASSWORD 'restaurante' CREATEDB;
CREATE DATABASE restaurante_db OWNER restaurante;
\q
```

> En Windows o macOS abre `psql` (o pgAdmin) con el usuario `postgres` y ejecuta las mismas dos sentencias.
> `CREATEDB` solo es necesario si vas a crear migraciones nuevas con `npm run db:migrate`.

Comprueba que la conexión funciona antes de seguir:

```bash
psql "postgresql://restaurante:restaurante@localhost:5432/restaurante_db" -c "SELECT 1"
```

### 3. Configurar y arrancar el backend

```bash
cd server
cp .env.example .env
```

Edita `server/.env`:

| Variable | Descripción |
|----------|-------------|
| `DATABASE_URL` | Cadena de conexión. Ajusta el usuario y la contraseña si usaste otros en el paso 2. |
| `JWT_SECRET` | **Obligatorio cambiarlo.** Genera uno con `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"` |
| `JWT_EXPIRES_IN` | Duración de la sesión (por defecto `8h`). |
| `CLIENT_URL` | URL del frontend, usada para CORS (por defecto `http://localhost:5173`). |
| `SEED_ADMIN_EMAIL` / `SEED_ADMIN_PASSWORD` | Credenciales del administrador inicial. |

Luego:

```bash
npm install
npm run db:deploy     # crea las tablas
npm run db:seed       # carga los datos iniciales
npm run dev           # API en http://localhost:4000/api
```

Verifica en otra terminal: `curl http://localhost:4000/api/health` debe responder `"status":"ok"`.

### 4. Configurar y arrancar el frontend

En **otra terminal**:

```bash
cd client
cp .env.example .env
npm install
npm run dev           # http://localhost:5173
```

Abre <http://localhost:5173>. En desarrollo, Vite redirige `/api` y `/socket.io` al backend, así que no hace falta configurar CORS.

### Usuarios de prueba

El seed crea un usuario por rol. En la pantalla de inicio de sesión (solo en desarrollo) puedes hacer clic en ellos para autocompletar.

| Rol | Correo | Contraseña |
|-----|--------|-----------|
| Administrador | admin@restaurante.com | `Admin123*` (o `SEED_ADMIN_PASSWORD`) |
| Mesero | mesero@restaurante.com | `Demo123*` |
| Cocina | cocina@restaurante.com | `Demo123*` |
| Cliente | cliente@restaurante.com | `Demo123*` |

> ⚠️ Cambia o desactiva estas cuentas antes de usar el sistema en producción.

## Scripts disponibles

**Backend (`server/`)**

| Script | Descripción |
|--------|-------------|
| `npm run dev` | Servidor con recarga automática |
| `npm start` | Servidor en modo producción |
| `npm run db:deploy` | Aplicar migraciones pendientes |
| `npm run db:migrate` | Crear una migración nueva tras modificar `schema.prisma` |
| `npm run db:seed` | Cargar datos iniciales (se puede ejecutar varias veces) |
| `npm run db:reset` | ⚠️ Borrar la base, re-aplicar migraciones y seed |
| `npm run db:studio` | Explorar los datos en el navegador (Prisma Studio) |

**Frontend (`client/`)**

| Script | Descripción |
|--------|-------------|
| `npm run dev` | Servidor de desarrollo |
| `npm run build` | Compilación de producción en `dist/` |
| `npm run preview` | Servir la compilación localmente |

## Estructura del proyecto

```
Gestion_Restaurante/
├── server/                     # API REST + WebSockets
│   ├── prisma/                 # schema.prisma, migraciones y seed
│   ├── uploads/                # imágenes subidas (logo, platos)
│   └── src/
│       ├── config/             # entorno, Prisma, matriz de permisos
│       ├── middlewares/        # authenticate, authorize, validate, rateLimit, errorHandler
│       ├── modules/            # un módulo por funcionalidad (routes/controller/service/schema)
│       ├── sockets/            # Socket.io: autenticación, salas y eventos
│       ├── routes/             # montaje de módulos en /api
│       └── utils/
└── client/                     # SPA React
    └── src/
        ├── api/                # instancia de axios y clientes por módulo
        ├── components/         # ui/ (genéricos) y shared/
        ├── context/            # AuthContext
        ├── features/           # componentes por módulo
        ├── hooks/
        ├── layouts/            # PublicLayout, DashboardLayout
        ├── pages/
        ├── routes/             # AppRouter, ProtectedRoute, GuestRoute
        └── utils/              # constantes, navegación por rol, errores
```

## Roles y permisos

La matriz vive en [`server/src/config/permissions.js`](server/src/config/permissions.js) y es la única fuente de verdad del backend. El menú lateral del frontend se define en [`client/src/utils/navigation.js`](client/src/utils/navigation.js).

| Módulo | ADMIN | WAITER | KITCHEN | CLIENT |
|--------|:-----:|:------:|:-------:|:------:|
| Configuración | ✅ | — | — | — |
| Usuarios | ✅ | — | — | — |
| Menú (gestión) | ✅ | — | — | — |
| Menú (consulta) | ✅ | ✅ | ✅ | ✅ (público) |
| Mesas: configurar (crear/editar) | ✅ | — | — | — |
| Mesas: vista del salón, adicionar y cobrar mesa | ✅ | ✅ | — | 👁️ Solo lista de activas (para pedir) |
| Reservas | ✅ todas | ✅ todas | — | Propias |
| Pedidos (crear) | ✅ | ✅ | — | ✅ |
| Pedidos (consultar) | ✅ todos | ✅ todos | ✅ todos | Propios |
| Registrar cobro de pedidos | ✅ | ✅ | — | — |
| Pantalla de cocina (KDS) | ✅ | 👁️ Solo lectura | ✅ | — |
| Caja (apertura/cierre) y reportes | ✅ | — | — | — |

### Estados de pedidos por rol

```
PENDING ──► IN_PREPARATION ──► READY ──► DELIVERED ──► PAID
   │              │              │
   └──────────────┴──────────────┴──► CANCELLED
```

| Rol | Transiciones permitidas |
|-----|------------------------|
| KITCHEN | `PENDING → IN_PREPARATION`, `IN_PREPARATION → READY` |
| WAITER | `READY → DELIVERED`, `DELIVERED → PAID` (cobro), `PENDING → CANCELLED` |
| ADMIN | Todas |
| CLIENT | Ninguna |

> Registrar un cobro (`→ PAID`) requiere un turno de caja abierto; solo el administrador abre y cierra la caja.

## API

Todas las respuestas usan el formato `{ "data": … }`, o `{ "error": { "message", "details" } }` cuando algo falla. Las rutas privadas requieren la cabecera `Authorization: Bearer <token>`.

### Autenticación — `/api/auth`

| Método | Ruta | Acceso | Descripción |
|--------|------|--------|-------------|
| POST | `/register` | Público | Crea una cuenta `CLIENT` y devuelve `{ user, token }` |
| POST | `/login` | Público | Devuelve `{ user, token }` |
| GET | `/me` | Autenticado | Usuario actual |
| PATCH | `/me` | Autenticado | Actualiza `name` y `phone` |
| PATCH | `/me/password` | Autenticado | `{ currentPassword, newPassword }` |

### Usuarios — `/api/users` (solo ADMIN)

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/` | Lista paginada. Query: `search`, `role`, `status` (`active`/`inactive`/`all`), `page`, `limit` |
| GET | `/:id` | Detalle |
| POST | `/` | Crear `{ name, email, password, role, phone? }` |
| PATCH | `/:id` | Editar cualquier campo; `{ isActive: true }` reactiva |
| DELETE | `/:id` | Desactivar (soft delete: se conserva el historial) |

### Configuración — `/api/settings`

| Método | Ruta | Acceso | Descripción |
|--------|------|--------|-------------|
| GET | `/` | Público | Configuración completa, con `businessHours` |
| PUT | `/` | ADMIN | `restaurantName`, `address`, `phone`, `email`, `currency`, `taxRate`, `suggestedTipRate`, `timezone`, `reservationSlotMinutes`, `maxGuestsPerSlot` |
| PUT | `/hours` | ADMIN | `{ hours: [{ dayOfWeek (0 = domingo), isOpen, openTime, closeTime }] }` con los 7 días |
| POST | `/logo` | ADMIN | `multipart/form-data`, campo `logo` (JPG/PNG/WEBP, máx. 2 MB) |
| DELETE | `/logo` | ADMIN | Quita el logo |

### Menú — `/api/menu`, `/api/categories`, `/api/products`

| Método | Ruta | Acceso | Descripción |
|--------|------|--------|-------------|
| GET | `/api/menu` | Público | Categorías visibles con sus platos activos |
| GET | `/api/categories` | ADMIN | Todas las categorías, con `productCount` |
| POST / PATCH / DELETE | `/api/categories[/:id]` | ADMIN | `{ name, description?, sortOrder?, isActive? }`. Solo se elimina si no tiene platos |
| GET | `/api/products` | ADMIN | Query: `categoryId`, `search`, `status` (`active`/`archived`/`all`) |
| POST / PATCH | `/api/products[/:id]` | ADMIN | `{ name, description?, price, categoryId, isAvailable?, isActive? }` |
| DELETE | `/api/products/:id` | ADMIN | Elimina; si el plato ya se vendió lo archiva (`{ archived: true }`) |
| POST / DELETE | `/api/products/:id/image` | ADMIN | Subir (campo `image`) o quitar la imagen |

### Mesas — `/api/tables`

| Método | Ruta | Acceso | Descripción |
|--------|------|--------|-------------|
| GET | `/` | ADMIN (todas) · WAITER, CLIENT (activas) | Lista de mesas |
| POST / PATCH | `/[:id]` | ADMIN | `{ number, capacity, isActive? }` |
| DELETE | `/:id` | ADMIN | Solo si no tiene pedidos ni reservas; si no, desactívala |
| POST | `/:id/pay` | ADMIN, WAITER | Cobra toda la cuenta de la mesa `{ paymentMethod: CASH \| TRANSFER }`. Requiere caja abierta y que la cocina haya terminado; los pedidos «Listos» se marcan entregados |

### Pedidos — `/api/orders`

| Método | Ruta | Acceso | Descripción |
|--------|------|--------|-------------|
| GET | `/` | ADMIN, WAITER, KITCHEN (todos) · CLIENT (propios) | Query: `status` (p. ej. `PENDING,READY`), `from` (fecha ISO), `limit` |
| GET | `/:id` | Ídem | Detalle con platos, mesa, mesero y cliente |
| POST | `/` | ADMIN, WAITER, CLIENT | `{ type: DINE_IN \| TAKEAWAY \| DELIVERY, tableId?, customerName?, notes?, items: [{ productId, quantity, notes? }] }`. Domicilio: `customerPhone`, `deliveryAddress`, `deliveryFee` (siempre se suma al total). `customerName` obligatorio |
| PATCH | `/:id/status` | ADMIN, WAITER, KITCHEN | `{ status }`, según las transiciones del rol |
| POST | `/:id/pay` | ADMIN, WAITER | `{ paymentMethod: CASH \| TRANSFER, payCourierFromCash? }`. Requiere caja abierta. Sin propina. En domicilios registra el pago al domiciliario como salida de caja (por defecto) |

### Caja — `/api/cash-register`

| Método | Ruta | Acceso | Descripción |
|--------|------|--------|-------------|
| GET | `/current` | ADMIN (con totales) · WAITER (solo si está abierta) | Turno abierto o `null` |
| POST | `/open` | ADMIN | `{ openingAmount, notes? }` |
| POST | `/withdrawals` | ADMIN | Salida de efectivo `{ amount, categoryId, description }` (no puede superar el efectivo disponible) |
| DELETE | `/withdrawals/:id` | ADMIN | Elimina una salida registrada por error (solo con el turno abierto; no aplica a pagos de domicilio) |
| POST | `/close` | ADMIN | `{ countedCash, notes? }`: calcula totales, salidas y diferencia |
| GET | `/history` | ADMIN | Últimos cierres |

### Gastos — `/api/expenses` y `/api/expense-categories` (solo ADMIN)

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/expenses?month=YYYY-MM` | Gastos del mes. Filtros: `categoryId`, `source` (`all`/`cash`/`manual`) |
| GET | `/api/expenses/summary?month=YYYY-MM` | Total del mes y total por categoría |
| POST / PATCH / DELETE | `/api/expenses[/:id]` | `{ categoryId, amount, description, periodMonth, paidDate?, paymentMethod? }`. Las salidas de caja solo se gestionan desde Caja |
| GET / POST / PATCH / DELETE | `/api/expense-categories[/:id]` | `{ name, isActive?, sortOrder? }`. «Domicilios» es del sistema; una categoría con gastos solo se desactiva |

### Reportes — `/api/reports` (solo ADMIN)

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/daily?date=YYYY-MM-DD` | Día (por defecto hoy): total, pedidos, ticket promedio, plato más vendido, método predominante, ventas por hora, salidas |
| GET | `/monthly?month=YYYY-MM` | Mes (por defecto el actual): ventas por día, total, día con más y menos ventas, comparación con el mes anterior, platos, métodos de pago, gastos por categoría y ganancia |

### Tiempo real (Socket.io)

Conexión: `io(URL, { auth: { token: '<JWT>' } })`. Eventos que emite el servidor:

| Evento | Destinatarios | Contenido |
|--------|---------------|-----------|
| `order:created` | Personal (ADMIN, WAITER, KITCHEN) y el cliente dueño | Pedido completo |
| `order:updated` | Ídem | Pedido completo tras un cambio de estado o cobro |

### Otros

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/health` | Estado de la API y de la base de datos |
| GET | `/uploads/...` | Imágenes subidas (logo y platos) |

## Solución de problemas

| Problema | Solución |
|----------|----------|
| `P1000: Authentication failed` | El usuario o la contraseña de `DATABASE_URL` no coinciden con los creados en PostgreSQL. |
| `P1001: Can't reach database server` | PostgreSQL no está corriendo (`sudo systemctl start postgresql`) o el puerto no es 5432. |
| `Variables de entorno inválidas` al arrancar | Falta `server/.env` o `JWT_SECRET` tiene menos de 32 caracteres. |
| La página muestra «No se pudo conectar con el servidor» | El backend no está corriendo en el puerto 4000. |

## Licencia

Distribuido bajo la licencia **MIT**. Consulta [LICENSE.txt](LICENSE.txt).
# RestauranteRustikko
