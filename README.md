# FerreteriaLujan-UI

Interfaz web de gestión para **Ferretería Luján**. Es el frontend de [FerreteriaLujan-API](https://github.com/DiogoDelvalle/FerreteriaLujan-API) desarrollado con React.js y permite operar los módulos de **ventas, compras e inventario** desde el navegador, con inicio de sesión y permisos por rol.

## Qué hace

- **Login** contra `ms-seguridad` (JWT con renovación automática del token). Sin sesión solo se accede a `/login`.
- **Ventas**
  - Venta directa con buscador de productos con sugerencias.
  - Presupuestos (alta con detalle, listado y visualización).
  - Facturas de venta (listado y vista de detalle).
  - Alta de clientes desde un modal.
- **Compras**
  - Facturas de compra (alta en dos pasos: cabecera y líneas de detalle).
  - Proveedores (alta y edición).
  - Pedidos y seguimiento (hoy con datos de prueba: el backend todavía no tiene entidad de pedidos).
- **Inventario**
  - Productos, depósitos y stock.
  - Transferencias entre depósitos.
- **Reportes**: pantalla reservada, aún sin contenido.
- Páginas de error (400 / 500) y botones de borrado visibles solo para el rol **ADMIN**. El backend es quien realmente valida el permiso.

## Tecnologías

- **React 19** + **React Router 7**
- **Vite 8** (servidor de desarrollo con proxy a `/api`)
- **Axios** (cliente HTTP único con interceptores)
- **oxlint** para linting
- CSS propio (`src/styles/theme.css`)

## Arquitectura del frontend

La UI no habla nunca con un microservicio ni con la base de datos: todo pasa por el **API Gateway** (`/api`).

```
src/
├── gateway/     # client.js (axios + refresh de token), routes.js (URLs), session.js, roles.js
├── dto/         # mapeo DTO del backend <-> modelo de la UI (ventas, compras, inventario, seguridad, page)
├── services/    # recurso.js (CRUD genérico) + ventasService, comprasService, inventarioService, seguridadService
├── components/  # Layout, navegación, modales (FormModal, DetalleModal, ConfirmarEliminar),
│                # BuscadorSugerencias, Paginacion y factura/ (formulario y lista reutilizables)
├── pages/       # Login, Main, ErrorPage y modulos/{ventas,compras,inventario}
└── hooks/       # useCargar
```

- **`gateway/client.js`**: agrega el `Bearer` token a cada petición y, ante un 401, hace **un único refresh** aunque haya varias peticiones fallando a la vez; si no puede renovar, cierra la sesión y vuelve al login.
- **`services/recurso.js`**: CRUD genérico (`listar`, `listarTodo`, `buscar`, `obtener`, `crear`, `actualizar`, `eliminar`) sobre cualquier recurso del gateway, con conversores `desde`/`hacia` entre DTO y modelo de la UI.
- **`dto/`**: aísla a la UI de los nombres de campo del backend.

## Rutas de la aplicación

| Ruta | Pantalla |
|---|---|
| `/login` | Inicio de sesión |
| `/` | Menú principal |
| `/ventas`, `/ventas/venta-directa`, `/ventas/presupuestos`, `/ventas/facturas` | Módulo de ventas |
| `/compras/pedidos`, `/compras/facturas`, `/compras/facturas/nueva`, `/compras/proveedores` | Módulo de compras |
| `/inventario/productos`, `/inventario/transferencias`, `/inventario/transferencias/nueva` | Módulo de inventario |
| `/reportes` | Reportes (vacío por ahora) |

## Equipo y contribuciones

Proyecto desarrollado en equipo en el marco de FIUNI.

### Diogo Delvalle (`diogodelvalleFiuni`)

- **Base del proyecto** (29/10/2026): estructura inicial con Vite + React, enrutamiento, login, layout y tema visual.
- **Capa de conexión con el backend** (10/11/2026): diseñó la arquitectura `gateway/` + `dto/` + `services/` que usa toda la UI: cliente axios con renovación de token, tabla de rutas, manejo de sesión, mapeo de DTOs y el servicio CRUD genérico `recurso.js`. Reemplazó la primera versión de `src/api/`.
- **Módulo de ventas completo**:
  - Presupuestos y su modal de alta (`Presupuestos.jsx`, `ModalNuevoPresupuesto.jsx`).
  - Venta directa y facturas de venta, conectadas a la API con `ventasService.js` (11–13/11/2026).
  - Vista de detalle de factura de venta (`VerFacturaVenta.jsx`) y nuevo modal de detalle.
- **Buscador de productos con sugerencias** (`BuscadorSugerencias.jsx`), usado en la venta directa, con su ajuste del máximo de sugerencias.
- **Mejoras de interfaz**: cambios en la barra de navegación, iconos, formato de visualización de facturas de venta y eliminación del scroll global.
- **Proxy de desarrollo** en `vite.config.js` que quita la cabecera `Origin`, en conjunto con el cambio de CORS que hizo en el API Gateway del backend.

### Juan Francken (`JuanFrancken`)

- Módulo de **inventario**: pantallas de productos y transferencias, correcciones de UI y conexión con el backend.
- Componentes reutilizables: `FormModal`, `ConfirmarEliminar` y modal de detalle.
- Páginas de error y limpieza de navegación.

### Nicole Quiñonez (`nicole8086`)

- Módulo de **compras**: facturas de compra, formulario reutilizable de facturas, edición y proveedores.
- Documentación de conexión con el backend (`docs/CONEXION_BACKEND.md`).
- Logo y ajustes del layout.
