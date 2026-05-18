# ABP - FarmaExpres

## 1. Identificacion del proyecto

**Nombre del proyecto:** FarmaExpres  
**Tipo de sistema:** Aplicacion web para gestion de inventario farmaceutico  
**Arquitectura implementada:** Frontend web con React y backend basado en microservicios  
**Roles implementados:** Administrador, Farmaceutico y Auditor

FarmaExpres es un sistema orientado a la gestion operativa de una farmacia. La solucion implementada permite administrar usuarios, autenticar el acceso, gestionar medicamentos, controlar entradas y salidas de inventario, consultar movimientos, visualizar alertas, generar reportes, revisar indicadores por rol y ejecutar funciones de auditoria.

Este ABP se construye a partir de lo implementado en los repositorios actuales de frontend y backend del proyecto.

## 2. Problema abordado

En la operacion de una farmacia se requiere controlar medicamentos, existencias, lotes, vencimientos, entradas, salidas y usuarios responsables de cada accion. Sin un sistema centralizado se dificulta mantener trazabilidad, controlar accesos por rol, detectar productos vencidos o agotados, consultar el historial de movimientos y revisar inconsistencias operativas.

FarmaExpres aborda esta necesidad mediante una aplicacion web conectada a servicios backend especializados, con control de acceso, gestion de inventario, alertas, reportes y auditoria.

## 3. Objetivo general

Desarrollar un sistema web distribuido para la gestion de inventario farmaceutico, permitiendo el control de usuarios, medicamentos, stock, movimientos, alertas, reportes y auditoria mediante una arquitectura de microservicios y una interfaz organizada por roles.

## 4. Objetivos especificos

- Implementar autenticacion de usuarios con tokens JWT.
- Gestionar usuarios y roles desde la interfaz administrativa.
- Registrar, consultar, actualizar y desactivar medicamentos.
- Controlar entradas y salidas de inventario.
- Consultar el historial de movimientos.
- Visualizar alertas de medicamentos vencidos, proximos a vencer, con bajo stock y agotados.
- Generar reportes de inventario, movimientos, vencimientos, bajo stock y actividad por usuario.
- Implementar un modulo de auditoria para revisar historicos, inconsistencias, observaciones y metricas.
- Centralizar el acceso a los servicios mediante un API Gateway.
- Versionar la base de datos mediante migraciones Liquibase.

## 5. Alcance implementado

### 5.1 Frontend

El frontend esta implementado como una aplicacion React con Vite. La estructura esta organizada por modulos funcionales:

- Autenticacion.
- Dashboard.
- Gestion de usuarios.
- Gestion de medicamentos.
- Entradas de inventario.
- Salidas de inventario.
- Movimientos.
- Alertas.
- Reportes.
- Control de stock.
- Auditoria.
- Layout y componentes compartidos.

La navegacion se adapta al rol autenticado:

| Rol | Modulos visibles implementados |
| --- | --- |
| Administrador | Dashboard, Medicamentos, Usuarios, Movimientos, Reportes, Alertas, Control de stock |
| Farmaceutico | Dashboard, Inventario, Entradas, Salidas, Alertas, Control de stock |
| Auditor | Dashboard, Inventario, Movimientos, Reportes, Auditoria |

### 5.2 Backend

El backend esta implementado con una arquitectura de microservicios:

| Servicio | Responsabilidad implementada |
| --- | --- |
| `api-gateway` | Punto de entrada centralizado, seguridad JWT, enrutamiento y fallbacks |
| `auth-service` | Autenticacion, refresh token, logout, usuarios, roles y bitacora |
| `inventory-service` | Medicamentos, lotes, movimientos, entradas, salidas, reportes y alertas de inventario |
| `alert-service` | Alertas de inventario consumiendo informacion del servicio de inventario |
| `audit-service` | Historial de auditoria, inconsistencias, observaciones, metricas y casos manuales |

La persistencia usa PostgreSQL y migraciones Liquibase separadas por dominio: autenticacion, inventario y auditoria.

## 6. Funcionalidades implementadas

### 6.1 Autenticacion y sesion

El sistema permite iniciar sesion mediante correo y contrasena. El backend genera tokens JWT y refresh tokens. El frontend conserva la sesion, protege rutas privadas y redirige usuarios no autenticados al login.

Endpoints implementados principales:

- `POST /api/auth/login`
- `POST /api/auth/refresh`
- `POST /api/auth/logout`

### 6.2 Gestion de usuarios

El sistema permite consultar usuarios, crear usuarios, actualizar datos, cambiar contrasena, bloquear, desbloquear y desactivar usuarios. Esta funcionalidad esta disponible en el frontend para el rol Administrador.

Endpoints implementados principales:

- `GET /api/users`
- `POST /api/users`
- `PUT /api/users/{id}/update`
- `PUT /api/users/{id}/password`
- `PUT /api/users/{id}/block`
- `PUT /api/users/{id}/unlock`
- `DELETE /api/users/{id}`

### 6.3 Gestion de medicamentos e inventario

El sistema permite administrar medicamentos, consultar productos activos, consultar todos los productos, ver detalle por identificador, registrar medicamentos, actualizar medicamentos y realizar eliminacion logica.

Tambien se implementa consulta por lotes, resumen de inventario activo, tabla de inventario activo y vista FEFO del inventario.

Endpoints implementados principales:

- `GET /api/products`
- `GET /api/products/all`
- `GET /api/products/{id}`
- `GET /api/products/active`
- `GET /api/products/active-table`
- `GET /api/products/active-summary`
- `GET /api/products/fefo-snapshot`
- `POST /api/products`
- `PUT /api/products/{id}`
- `DELETE /api/products/{id}`
- `GET /api/products/{productId}/batches`
- `POST /api/products/{productId}/batches`

### 6.4 Entradas y salidas de inventario

El sistema permite registrar entradas y salidas de inventario desde el frontend. Las entradas aumentan existencias y las salidas descuentan stock. El backend cuenta con endpoints diferenciados para ambos movimientos.

Endpoints implementados principales:

- `POST /api/movements/entries`
- `POST /api/movements/exits`
- `POST /api/movements/consume-fefo`

### 6.5 Historial de movimientos

El sistema permite consultar movimientos de inventario, ver movimientos por identificador, filtrar por tipo, fecha y usuario, consultar entradas, salidas y movimientos actualizados.

Endpoints implementados principales:

- `GET /api/movements`
- `GET /api/movements/{id}`
- `GET /api/movements/filter-by-user`
- `GET /api/movements/entrance`
- `GET /api/movements/exit`
- `GET /api/movements/updated`
- `PATCH /api/movements/{id}/audit-status`

### 6.6 Alertas

El sistema implementa un centro de alertas con secciones para:

- Medicamentos vencidos.
- Medicamentos proximos a vencer.
- Medicamentos con bajo stock.
- Productos agotados.

El frontend muestra conteo de alertas en el menu lateral para los roles con acceso. El backend expone alertas desde inventario y tambien existe un microservicio de alertas en Node.js.

Endpoints implementados principales en inventario:

- `GET /api/inventory/alerts/low-stock`
- `GET /api/inventory/alerts/out-of-stock`
- `GET /api/inventory/alerts/expired`
- `GET /api/inventory/alerts/expiring-soon`
- `GET /api/inventory/alerts/expiring-range`

### 6.7 Reportes

El frontend implementa pestañas de reportes para:

- Inventario actual.
- Movimientos.
- Proximos a vencer.
- Bajo stock.
- Por usuario.

Tambien existen utilidades frontend para exportacion de reportes.

Endpoints implementados principales:

- `GET /api/reports/inventory-batches`
- `GET /api/reports/movements-batches`
- `GET /api/inventory/reports/expiring-products`
- `GET /api/inventory/reports/batches`
- `GET /api/movements/report/users-activity`
- `GET /api/products/low-stock`
- `GET /api/products/low-stock/critical`
- `GET /api/products/low-stock/alert`
- `GET /api/products/out-of-stock`

### 6.8 Control de stock

El sistema cuenta con modulo de control de stock accesible para Administrador y Farmaceutico. Este modulo consume informacion del inventario para revisar estado de productos, niveles de stock y productos en condicion critica o de alerta.

### 6.9 Dashboard

El frontend cuenta con dashboard disponible para los tres roles implementados. Este modulo resume informacion operativa del sistema segun la sesion activa y datos disponibles desde los servicios.

### 6.10 Auditoria

El sistema cuenta con un modulo de auditoria para el rol Auditor. Permite consultar historial, inconsistencias, observaciones y metricas. Tambien permite recalcular auditoria, crear casos manuales, actualizar notas, cambiar estados y quitar marcas manuales.

Endpoints implementados principales:

- `GET /api/audit/history`
- `GET /api/audit/inconsistencies`
- `GET /api/audit/observations`
- `GET /api/audit/metrics`
- `POST /api/audit/recalculate`
- `POST /api/audit/cases/manual`
- `PATCH /api/audit/cases/{id}/note`
- `PATCH /api/audit/cases/{id}/status`
- `DELETE /api/audit/cases/{id}/manual-flag`

### 6.11 Estado de servicios

Los servicios backend implementan endpoints de estado para validar disponibilidad.

Endpoints implementados:

- `GET /status` en `api-gateway`
- `GET /status` en `auth-service`
- `GET /status` en `inventory-service`
- `GET /status` en `audit-service`
- `GET /status` en `alert-service`

## 7. Arquitectura implementada

La solucion usa una arquitectura distribuida basada en servicios separados. El frontend consume el backend mediante un gateway o rutas relativas configuradas por ambiente.

Componentes principales:

- Frontend React/Vite.
- API Gateway en Spring Boot.
- Servicios Spring Boot para autenticacion, inventario y auditoria.
- Servicio Node.js/Express para alertas.
- PostgreSQL como base de datos.
- Liquibase para versionamiento de esquema.
- Docker Compose para levantar el entorno.

## 8. Tecnologias implementadas

### Frontend

- React.
- Vite.
- JavaScript ES Modules.
- React Router.
- Axios.
- Tailwind CSS.
- ESLint.
- Docker y Nginx.

### Backend

- Java 17.
- Spring Boot.
- Spring Security.
- Spring Data JPA.
- Spring Cloud Gateway.
- Resilience4j en API Gateway.
- JWT con JJWT.
- PostgreSQL.
- Liquibase.
- Node.js.
- Express.
- Docker Compose.

## 9. Evidencias disponibles en los repositorios

### Backend

En el repositorio backend existen:

- Codigo fuente de los microservicios.
- Dockerfiles por servicio.
- `docker-compose.yml` general.
- Migraciones Liquibase por dominio.
- Pruebas para servicios Spring Boot.
- Pruebas para `alert-service`.
- Documentacion de ADR.
- Documentacion de cambios por historias de usuario.

### Frontend

En el repositorio frontend existen:

- Codigo fuente modular por funcionalidad.
- Servicios de consumo de API.
- Control de rutas protegidas.
- Control de acceso por rol.
- Documentacion funcional por HU.
- Capturas de evidencia por modulo.
- Dockerfile, configuracion Nginx y Docker Compose.

## 10. Resultado del proyecto

El resultado implementado es una aplicacion web funcional para gestion farmaceutica con separacion entre interfaz de usuario y servicios backend. El sistema permite operar inventario, usuarios, movimientos, alertas, reportes, control de stock y auditoria, manteniendo acceso diferenciado por rol y comunicacion centralizada a traves de servicios backend.

