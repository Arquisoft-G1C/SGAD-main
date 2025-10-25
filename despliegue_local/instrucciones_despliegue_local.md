
# ⚙️ Guía de Despliegue Local – SGAD (Sistema de Gestión de Árbitros y Designaciones)
### Grupo Arquisoft G1C – Prototype 2

---

## 🧱 1. Requisitos previos

| Requisito | Versión recomendada |
|------------|---------------------|
| Docker | ≥ 24.0 |
| Docker Compose | ≥ 2.24 |
| Node.js | ≥ 20.0 |
| Python | ≥ 3.10 |
| Java | ≥ 21 |
| Git | Última estable |

---

## 🗂️ 2. Estructura general del ecosistema SGAD

```
Arquisoft-G1C/
├── SGAD-main/
├── sgad-api-gateway/
├── sgad-frontend/
├── sgad-mobile/
├── sgad-match-management/
├── sgad-referee-management/
├── sgad-availability-service/
├── sgad-auth-service/
└── sgad-infraestructure/
```

---

## 🚀 3. Despliegue completo con Docker Compose

### 3.1. Clonar el proyecto
```bash
git clone https://github.com/Arquisoft-G1C/SGAD-main
cd SGAD-main
```

### 3.2. Configurar variables de entorno
```bash
cp .env.example .env
```
Editar `.env` con valores locales (PostgreSQL, MongoDB, Redis).

---

### 3.3. Levantar el sistema
```bash
docker compose up --build
```

Servicios desplegados:
- PostgreSQL (5432)
- MongoDB (27017)
- Redis (6379)
- API Gateway (8080)
- Auth Service (3001)
- Match Service (8001)
- Referee Service (8002)
- Availability Service (8003)
- Frontend SSR (3000)

---

## 🌐 4. Accesos principales

| Componente | URL local | Descripción |
|-------------|-----------|--------------|
| Frontend SSR | http://localhost:3000 | Interfaz principal |
| API Gateway | http://localhost:8080 | Enrutador central |
| Auth Service | http://localhost:3001 | Autenticación JWT |
| Match Service | http://localhost:8001 | Gestión de partidos |
| Referee Service | http://localhost:8002 | Gestión de árbitros |
| Availability Service | http://localhost:8003 | Disponibilidad con scheduler |
| PostgreSQL | localhost:5432 | Base de datos SQL |
| MongoDB | localhost:27017 | Base de datos NoSQL |
| Redis | localhost:6379 | Mensajería/pub-sub |

---

## 🧩 5. Endpoints de prueba

```bash
# Login
curl -X POST http://localhost:3001/api/login -H "Content-Type: application/json" -d '{"email":"admin@unal.edu.co","password":"1234"}'

# Crear partido
curl -X POST http://localhost:8001/matches/ -H "Content-Type: application/json" -d '{"match_id":1,"date":"2025-10-25","referee_id":2}'

# Consultar árbitros
curl http://localhost:8002/referees/

# Revisar disponibilidad
curl http://localhost:8003/availability/
```

---

## 🧠 6. Monitoreo y logs

```bash
docker compose logs -f
docker ps
docker compose down
```

---

## 🧰 7. Ejecución por servicios

Infraestructura:
```bash
cd sgad-infraestructure
docker compose up -d
```

Auth Service:
```bash
cd sgad-auth-service
npm install
npm run dev
```

Match Service:
```bash
cd sgad-match-management
pip install -r requirements.txt
uvicorn app.main:app --reload --host 0.0.0.0 --port 8001
```

---

## ✅ 8. Verificación final

- Acceder a http://localhost:3000
- Comprobar rutas vía Gateway (8080)
- Logs de Redis deben mostrar eventos al crear partidos
- Scheduler activo en Availability Service

---

## 🏁 9. Limpieza

```bash
docker compose down -v
docker system prune -f
```

---

### 📘 Créditos
**Grupo Arquisoft G1C**  
Universidad Nacional de Colombia – 2025-II  
Proyecto: *SGAD – Prototype 2 (Advanced Architectural Structure)*
