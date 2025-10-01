# SGAD – Sistema de Gestión de Árbitros y Designaciones

![SGAD Logo](docs/logo.png)

---

## 📖 Descripción

El **SGAD (Sistema de Gestión de Árbitros y Designaciones)** es una plataforma web distribuida que facilita la **asignación imparcial de árbitros a partidos deportivos**, la **gestión administrativa** de árbitros y partidos, y el **control de disponibilidad y pagos**.  

Este proyecto corresponde al **Prototype 1** de la materia **Software Architecture (2025-II)**.

---

## 🎯 Objetivo del Prototipo

Construir un **prototipo vertical** basado en un diseño arquitectónico inicial que permita:  
- Validar el estilo arquitectónico escogido (**microservicios distribuidos con API Gateway**).  
- Demostrar el uso de **distintos lenguajes de programación**.  
- Integrar **bases de datos relacional y NoSQL**.  
- Probar el **despliegue en contenedores (Docker)**.  

---

## 📌 Requerimientos

### Funcionales
- Registro y consulta de árbitros.  
- Gestión de partidos.  
- Asignación de árbitros a partidos.  
- Autenticación básica de usuarios.  

### No Funcionales
- Arquitectura distribuida con microservicios.  
- 1 componente de presentación (frontend web).  
- 2 o más componentes de lógica (gestión de árbitros y partidos).  
- 2 componentes de datos (SQL y NoSQL).  
- 2 tipos de conectores HTTP (REST y API Gateway).  
- 2 lenguajes de programación (Python, JavaScript/TypeScript).  
- Despliegue contenerizado (Docker).  

---

## 🏗️ Estructuras Arquitectónicas

### 🔹 Vista General de la Arquitectura
![Vista General](docs/Vista%20General%20de%20la%20Arquitectura%20SGAD.JPG)  
Esta vista muestra todo el ecosistema del sistema **SGAD** en un solo diagrama:  
- **Actores externos**: Árbitro, Club/Equipo y Administrador.  
- **Frontend** como punto de acceso al sistema.  
- **API Gateway** como conector central de los microservicios.  
- **Microservicios**: gestión de partidos, gestión de árbitros y autenticación.  
- **Infraestructura**: bases de datos relacional (SQL) y NoSQL.  

---

### 🔹 Vista de Contexto
![Vista de Contexto](docs/Contexto%20del%20Sistema%20(vista%20de%20alto%20nivel).JPG)  
El sistema se ve como una **caja negra** y se representan las interacciones externas:  
- El **Árbitro** consulta designaciones.  
- El **Club/Equipo** solicita árbitros.  
- El **Administrador** gestiona partidos y recibe reportes y notificaciones.  

---

### 🔹 Vista C&C (Componentes y Conectores)
![Vista C&C](docs/Vista%20C&C%20(microservicios).JPG)  
Expone la estructura interna de SGAD como un sistema de **microservicios**:  
- `sgad-frontend` → interfaz de usuario.  
- `sgad-api-gateway` → enrutador de peticiones.  
- `sgad-match-management` → gestión de partidos.  
- `sgad-referee-management` → gestión de árbitros.  
- `sgad-auth-service` → autenticación básica.  
- `sgad-infrastructure` → soporte de bases de datos.  

---

### 🔹 Vista de Despliegue
![Vista de Despliegue](docs/Vista%20de%20Despliegue%20(Deployment).JPG)  
Describe cómo se despliegan los componentes en contenedores Docker:  
- Cada microservicio corre en su propio contenedor.  
- El **API Gateway** intermedia entre frontend y servicios.  
- **PostgreSQL** almacena información de partidos y autenticación.  
- **MongoDB** almacena la disponibilidad de árbitros y datos no estructurados.  

---

### 🔹 Caso de Uso – Asignación de Árbitro
![Caso de Uso](docs/Caso%20de%20uso%20(Asignación%20de%20árbitro).JPG)  
Representa un escenario dinámico: la **asignación de un árbitro a un partido**.  
1. El **Administrador** solicita la designación.  
2. El **Frontend** envía la solicitud al **API Gateway**.  
3. El **Match Service** valida el partido.  
4. El **Referee Service** consulta la disponibilidad en la BD.  
5. Se selecciona un árbitro disponible y se confirma al administrador.

---

## ⚙️ Decisiones Arquitectónicas

1. **Estilo**: microservicios distribuidos con API Gateway.  
2. **Lenguajes**: Python (servicios de negocio), JavaScript/TypeScript (frontend, gateway, auth).  
3. **Bases de datos**:  
   - Relacional (partidos, árbitros, pagos).  
   - NoSQL (disponibilidad, logs, historial).  
4. **Contenedores**: cada microservicio se despliega en Docker.  
5. **Conectores**: REST API entre microservicios y gateway.  

---
## 🎨 Estilos arquitectónicos (SGAD)

SGAD adopta un **estilo de microservicios** con **API Gateway** al frente. Cada capacidad de negocio (partidos, árbitros, autenticación) vive en su propio servicio, se despliega de forma independiente (Docker) y se comunica mediante **REST/HTTP** usando **JSON**.

### ¿Por qué microservicios y no monolito?

**Monolito (descartado para SGAD)**
- Pros: simplicidad inicial, una sola base de código, despliegue único.
- Contras: acoplamiento alto, una falla puede tumbar todo, escalar una parte obliga a escalar todo, barreras para usar múltiples lenguajes o motores de datos.

**Microservicios (elegido)**
- Se alinean con los **módulos de negocio** del dominio (partidos, árbitros, auth).
- Permiten **tecnologías heterogéneas** (p. ej., Python para lógica de negocio; Node.js en gateway/frontend).
- Habilitan **bases de datos policromas**: **PostgreSQL** para datos relacionales y **MongoDB** para disponibilidad/registros no estructurados.
- Soportan **despliegue/escala independiente** por demanda (p. ej., escalar Referee o Match sin tocar los demás).
- Encajan con la necesidad de **entregas incrementales** del curso (Prototype 1, 2, …).

### Ventajas principales

- **Escalabilidad independiente**: cada servicio escala según su carga.
- **Aislamiento de fallas**: una caída en un servicio no detiene a los demás.
- **Evolución tecnológica**: cambiar framework/BD en un servicio no afecta al resto.
- **Ciclos de despliegue más cortos**: se actualiza solo el servicio modificado.
- **Seguridad y gobierno centralizados** a través del **API Gateway** (autenticación, rate-limiting, logging).

### Trade-offs / Costes

- **Complejidad operacional**: más servicios → más orquestación/observabilidad.
- **Latencia de red**: llamadas entre servicios agregan hops.
- **Consistencia de datos**: se prefiere **consistencia eventual** entre servicios/BDs.
- **Pruebas de integración**: requieren entornos con varios contenedores (Compose).
- **Gestión de configuración**: variables de entorno por servicio (URLs, credenciales).

> Cómo mitigamos: Docker Compose para orquestación local, **API Gateway** para cross-cutting concerns, endpoints de health (`/health`) y convenciones de contratos REST.

### Comunicación (REST sobre HTTP)

- **Patrón**: **Frontend ⇆ API Gateway ⇆ Microservicios**.  
  El gateway enruta, autentica (JWT), aplica rate-limit y uniformiza respuestas/errores.
- **Formato**: `application/json` en request/response.
- **Verbos y rutas (ejemplos)**  
  - **Match Service**:  
    - `GET /matches`, `GET /matches/{id}`, `POST /matches`, `PUT /matches/{id}`, `DELETE /matches/{id}`
  - **Referee Service**:  
    - `GET /referees`, `GET /referees/{id}`, `POST /referees`, `PUT /referees/{id}`, `GET /referees/available`
  - **Auth Service**:  
    - `POST /auth/login`, `GET /auth/verify`, `GET /auth/profile`
- **Autenticación**: JWT en `Authorization: Bearer <token>`; el **gateway** valida y reenvía claims a los servicios.
- **Estados HTTP**: `200/201` ok, `400` solicitud inválida, `401/403` autenticación/autorización, `404` no encontrado, `409` conflicto, `500` error interno.
- **Salud/monitoring**:  
  - Gateway: `GET /health`, `GET /health/services`  
  - Servicios: `GET /health` (indica disponibilidad y conexión a su BD)
- **Descubrimiento/ruteo**: en local, nombres de servicio/puertos definidos en `docker-compose.yml` (p. ej., `match-service:8000`, `referee-service:8000`, `auth-service:3001`).

> Nota: Para notificaciones/colas futuras se considera **Redis** (cache/mensajería), manteniendo REST como canal principal para Prototype 1.

## 📂 Estructura del Repositorio

```
sgad-main/
│── README.md              # Documento principal
│── docker-compose.yml     # Orquestación de microservicios
│── .env.example           # Variables de entorno de ejemplo
│
├── docs/                  # Diagramas y documentación
│   ├── logo.png
│   ├── vista_general.jpg
│   ├── contexto.jpg
│   ├── cc.jpg
│   ├── deployment.jpg
│   └── caso_uso.jpg
│
└── scripts/               # Scripts de ayuda (opcional)
    ├── start.sh
    └── stop.sh
```

---

## 🚀 Despliegue Local

### 1. Clonar este repositorio
```bash
git clone https://github.com/Arquisoft-G1C/sgad-main.git
cd sgad-main
```

### 2. Configurar variables de entorno
Copiar `.env.example` a `.env` y ajustar credenciales (puertos, BDs, etc.).

### 3. Levantar el sistema
```bash
docker-compose up --build
```

### 4. Acceder al sistema
- Frontend: [http://localhost:3000](http://localhost:3000)  
- API Gateway: [http://localhost:8080](http://localhost:8080)  
- Documentación de APIs (Swagger): [http://localhost:8000/docs](http://localhost:8000/docs) *(ejemplo para servicios en FastAPI)*  

---

## 🔗 Microservicios relacionados

- [sgad-frontend](../sgad-frontend)  
- [sgad-api-gateway](../sgad-api-gateway)  
- [sgad-match-management](../sgad-match-management)  
- [sgad-referee-management](../sgad-referee-management)  
- [sgad-auth-service](../sgad-auth-service)  
- [sgad-infrastructure](../sgad-infrastructure)  

---

## 👥 Equipo

Grupo **Arquisoft G1C**  
- Wullfredo Javier Barco Godoy – wbarco@unal.edu.co  
- Nombre 2 – correo@unal.edu.co  
- Nombre 3 – correo@unal.edu.co  
- Nombre 4 – correo@unal.edu.co  

---

## 📅 Entregable

- **Entrega**: 1 de octubre de 2025 (23h59).  
- **Presentación**: 2 de octubre de 2025 (en clase).  
