# 🛡️ Implementación de *Network Segmentation Pattern* y *Secure Channel Pattern* en SGAD  
**Arquisoft – SGAD (Sistema de Gestión de Árbitros y Designaciones)**  
**Autor:** Javier Barco  
**Fecha:** 2025

---

## 📌 Introducción  
Con el objetivo de fortalecer la arquitectura del sistema SGAD, se implementaron dos tácticas de seguridad recomendadas en arquitecturas de microservicios distribuidas:

1. **Network Segmentation Pattern**  
2. **Secure Channel Pattern**

Ambas tácticas permiten mejorar el aislamiento, la protección de datos y la resistencia del sistema frente a ataques externos e internos.

Este documento detalla **qué se hizo**, **por qué se hizo** y **cómo quedó implementado** dentro del sistema SGAD.

---

# 🧱 1. Network Segmentation Pattern

## 🎯 Objetivo  
Dividir la red del sistema en **zonas o segmentos aislados**, de forma que:

- Los servicios externos **no puedan comunicarse directamente** con los microservicios internos.
- Evitar que un ataque comprometa toda la red.
- Limitar cuáles servicios pueden hablar con cuáles.

---

## 🗂️ 1.1 Segmentos de red creados  
Se agregaron **5 segmentos (networks) de Docker**:

```yaml
networks:
  sgad-network:
    driver: bridge

  edge-net:
    driver: bridge

  api-gw-net:
    driver: bridge

  backend-net:
    driver: bridge

  db-net:
    driver: bridge
```

### 🔹 Descripción de cada zona  
| Network | Propósito |
|--------|-----------|
| **edge-net** | Zona pública. Solo vive el reverse-proxy. Es la única entrada externa. |
| **api-gw-net** | Red privada para los API Gateways. No accesible desde afuera. |
| **backend-net** | Red interna de microservicios. Solo accesible para los gateways. |
| **db-net** | Red estrictamente interna para las bases de datos. |
| **sgad-network** | Red original, ahora usada solo para compatibilidad. |

---

## 🔐 1.2 Restricción de conexiones entre zonas

### 🔸 Reverse Proxy → API Gateway (permitido)
```yaml
networks:
  - edge-net
  - api-gw-net
```

### 🔸 API Gateway → Microservicios (permitido)
Gateways en:
```yaml
- api-gw-net
```

Microservicios en:
```yaml
- backend-net
```

### 🔸 Microservicios → Bases de datos (permitido)
```yaml
networks:
  - backend-net
  - db-net
```

---

## 🧩 1.3 Beneficios obtenidos

- ✔ Protección contra accesos directos a APIs internas  
- ✔ Aislamiento de bases de datos  
- ✔ Mínima superficie de ataque  
- ✔ Separación clara entre **zona pública – zona lógica – zona de datos**  
- ✔ Cumplimiento de buenas prácticas de NIST y OWASP  

---

# 🔒 2. Secure Channel Pattern

## 🎯 Objetivo  
Garantizar que **toda comunicación entre cliente y SGAD viaja cifrada**, evitando:

- Robo de datos  
- MITM  
- Alteración del tráfico  

---

## 🔐 2.1 Certificados TLS generados

```bash
openssl genrsa -out reverse-proxy.key 2048
openssl req -new -key reverse-proxy.key -out reverse-proxy.csr -subj "/CN=sgad.local"
openssl x509 -req -in reverse-proxy.csr -signkey reverse-proxy.key -out reverse-proxy.crt -days 730
```

---

## ⚙️ 2.2 Dockerfile actualizado

```dockerfile
COPY certs/reverse-proxy.crt /etc/nginx/ssl/
COPY certs/reverse-proxy.key /etc/nginx/ssl/
EXPOSE 80 443
```

---

## 🛠️ 2.3 nginx.conf HTTPS

```nginx
server {
    listen 443 ssl;
    ssl_certificate /etc/nginx/ssl/reverse-proxy.crt;
    ssl_certificate_key /etc/nginx/ssl/reverse-proxy.key;
}
```

---

# 🟢 3. Resultados Generales

### 🔰 Network Segmentation Pattern
- Aislamiento por zonas  
- Control de flujos de comunicación  
- Menor superficie de ataque  

### 🔐 Secure Channel Pattern
- Tráfico cifrado HTTPS  
- Protección contra MITM  
- Confidencialidad e integridad  

---

# 📝 Conclusión  
SGAD ahora cuenta con una arquitectura más sólida, con redes segmentadas y un canal seguro TLS, alineado con buenas prácticas profesionales de seguridad.

