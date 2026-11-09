# Documento Entregable: Auditoría de Seguridad y Remediación con Nuclei

**Alumno:** Elias Alain (GitHub: [Eliasvit1](https://github.com/Eliasvit1))  
**Profesor:** Alan Calvillo  
**Fecha de Entrega:** Día 14 (31 de Octubre de 2026)  
**Materia:** Seguridad en Aplicaciones y Sistemas Distribuidos  

---

## 1. Planteamiento de la Actividad y Caso de Ejecución

Siguiendo las instrucciones del profesor Alan Calvillo:
1. Se ejecutó un análisis de vulnerabilidades con **Nuclei** sobre los tres repositorios:
   * **Custom Dashboard (Frontend)**
   * **Backend API**
   * **LDAP Authentication Service**
2. Se guardaron los resultados en la carpeta dedicada `security-reports/` dentro de cada repositorio y se consolidaron en este paquete entregable.
3. Se generó un commit inicial de documentación:
   * Formato: `docs(security): add initial Nuclei security audit report`
4. Se procedió con la **remediación granular**: un commit independiente por cada vulnerabilidad solucionada, con mensajes en formato Conventional Commits en idioma inglés.

### Caso de Ejecución Nuclei Empleado (Opción A: Docker Bridge Network)
Ante la observación técnica respecto a conectividad y puertos inaccesibles:
> *"Posiblemente necesiten un bridge entre el contenedor de nuclei y lo que intentan analizar..."*

* **Solución aplicada:** Se desplegó el contenedor oficial de Nuclei conectado a la red bridge de Docker de los servicios (`--network ddos_labnet`).
* **Superación del bloqueo por Fail2ban:** Dado que el entorno cuenta con protección activa contra ráfagas (Fail2ban `http-flood`), al recibir ráfagas de escaneo bloqueó la IP del contenedor de Nuclei (`172.29.0.2`). Se configuró la lista blanca en Fail2ban para la subred bridge (`172.29.0.0/16`), permitiendo un escaneo exhaustivo sin caídas ni falsos rechazos por puerto.

---

## 2. Enlaces a los Repositorios en GitHub y Commits Granulares

### 1. Repositorio: Custom Dashboard (Frontend)
* **URL GitHub:** [https://github.com/Eliasvit1/security-frontend](https://github.com/Eliasvit1/security-frontend)
* **Carpeta de Reportes:** [security-reports](https://github.com/Eliasvit1/security-frontend/tree/main/security-reports)
* **Historial de Commits:**
  1. [`95341cb`](https://github.com/Eliasvit1/security-frontend/commit/95341cb) - `docs(security): add initial Nuclei security audit report`
  2. [`09e273e`](https://github.com/Eliasvit1/security-frontend/commit/09e273e) - `fix(auth): sanitize user input to prevent XSS vulnerability`
  3. [`c2187d2`](https://github.com/Eliasvit1/security-frontend/commit/c2187d2) - `fix(dashboard): enforce security headers and suppress server tokens`
  4. [`42d1c7b`](https://github.com/Eliasvit1/security-frontend/commit/42d1c7b) - `refactor(frontend): migrate to React SPA with multi-page session persistence`

### 2. Repositorio: Backend API
* **URL GitHub:** [https://github.com/Eliasvit1/security-api](https://github.com/Eliasvit1/security-api)
* **Carpeta de Reportes:** [security-reports](https://github.com/Eliasvit1/security-api/tree/main/security-reports)
* **Historial de Commits:**
  1. [`70bb1b0`](https://github.com/Eliasvit1/security-api/commit/70bb1b0) - `docs(security): add initial Nuclei security audit report`
  2. [`e07f97b`](https://github.com/Eliasvit1/security-api/commit/e07f97b) - `fix(api): restrict CORS origins and disable wildcard credentials`
  3. [`405f695`](https://github.com/Eliasvit1/security-api/commit/405f695) - `fix(api): implement rate limiting on sensitive endpoints`
  4. [`3508863`](https://github.com/Eliasvit1/security-api/commit/3508863) - `fix(api): disable unauthenticated OpenAPI and Swagger documentation endpoints`
  5. [`831b6cc`](https://github.com/Eliasvit1/security-api/commit/831b6cc) - `fix(api): enforce security headers and suppress server tokens`
  6. [`4ca4bc6`](https://github.com/Eliasvit1/security-api/commit/4ca4bc6) - `fix(cors): authorize React frontend development origins`

### 3. Repositorio: LDAP Service
* **URL GitHub:** [https://github.com/Eliasvit1/FastApi-Openldap](https://github.com/Eliasvit1/FastApi-Openldap)
* **Carpeta de Reportes:** [security-reports](https://github.com/Eliasvit1/FastApi-Openldap/tree/main/security-reports)
* **Historial de Commits:**
  1. [`731a8b3`](https://github.com/Eliasvit1/FastApi-Openldap/commit/731a8b3) - `docs(security): add initial Nuclei security audit report`
  2. [`e75e60a`](https://github.com/Eliasvit1/FastApi-Openldap/commit/e75e60a) - `fix(ldap): disable anonymous binding and enforce authentication`
  3. [`afa2fe4`](https://github.com/Eliasvit1/FastApi-Openldap/commit/afa2fe4) - `fix(auth): implement rate limiting on sensitive login endpoint`
  4. [`2ef22fc`](https://github.com/Eliasvit1/FastApi-Openldap/commit/2ef22fc) - `fix(auth): restrict CORS origins and disable wildcard credentials`
  5. [`c93f39c`](https://github.com/Eliasvit1/FastApi-Openldap/commit/c93f39c) - `fix(auth): disable unauthenticated OpenAPI and Swagger documentation endpoints`
  6. [`36e6d0c`](https://github.com/Eliasvit1/FastApi-Openldap/commit/36e6d0c) - `fix(auth): enforce security headers and suppress server tokens`
  7. [`31a907f`](https://github.com/Eliasvit1/FastApi-Openldap/commit/31a907f) - `fix(cors): authorize React frontend development origins`

---

## 3. Matriz de Vulnerabilidades Encontradas vs Parches Aplicados

| # | Repositorio | Vulnerabilidad Detectada por Nuclei | Riesgo / Vector | Parche Aplicado | Commit |
|---|---|---|---|---|---|
| 1 | **Frontend** | Falta de saneamiento en input de usuario | XSS / DOM Injection | Validación por regex y codificación de entidades HTML (`sanitizeInput`, `isValidUsername`). | `09e273e` |
| 2 | **Frontend** | Banner de Nginx y falta de cabeceras de seguridad | Information disclosure / Clickjacking | Se configuró `server_tokens off;`, `CSP`, `X-Frame-Options: DENY`, `nosniff`, `Referrer-Policy`. | `c2187d2` |
| 3 | **Backend** | `cors-misconfig:arbitrary-origin` | Robo de datos cross-site con credenciales | Se reemplazó wildcard `*` con lista blanca explícita de orígenes autorizados. | `e07f97b` |
| 4 | **Backend** | Ausencia de Rate Limiting en `/api/data` | DoS / saturación de recursos | Implementación de Rate Limiting por IP (ventana deslizante de 30 req/min) con retorno `429 Too Many Requests`. | `405f695` |
| 5 | **Backend** | Exposición pública de Swagger (`/docs`, `/redoc`, `openapi.json`) | Enumeración de endpoints internos | Desactivación de rutas de documentación en entorno de producción. | `3508863` |
| 6 | **Backend** | Cabeceras de seguridad faltantes y versión de servidor | MIME sniffing / Divulgación de versión | Middleware HTTP en FastAPI con cabeceras estrictas y Nginx con `server_tokens off;`. | `831b6cc` |
| 7 | **LDAP** | Permisión de Anonymous Binding en OpenLDAP | Enumeración no autenticada de directorio | Modificación slapd (`00-disable-anon.ldif` con `bind_anon` denegado y `olcRequires: authc`) y validación estricta de credenciales no vacías. | `e75e60a` |
| 8 | **LDAP** | Ausencia de Rate Limiting en `/login` | Ataques de fuerza bruta y credential stuffing | Throttling por IP en el endpoint de autenticación con bloqueo a 15 intentos/minuto (`429`). | `afa2fe4` |
| 9 | **LDAP** | CORS wildcard con credenciales | Acceso no autorizado a autenticación | Restricción estricta de orígenes autorizados en FastAPI CORS middleware. | `2ef22fc` |
| 10 | **LDAP** | Swagger y OpenAPI expuestos en microservicio Auth | Fuga de esquema de autenticación | Inhabilitación de Swagger UI y OpenAPI schema en producción. | `c93f39c` |
| 11 | **LDAP** | Falta de cabeceras de seguridad y divulgación de versión | Exposición de cabeceras HTTP | Integración de middleware de cabeceras de defensa en profundidad y Nginx hardening. | `36e6d0c` |

---

## 4. Pruebas de Validación Post-Remediación

1. **Re-escaneo con Nuclei:** Un segundo escaneo sobre las plantillas `cors`, `swagger`, `openapi` y `redoc` concluyó con **0 coincidencias (0 matches found)**.
2. **Validación de Cabeceras:** Todas las respuestas HTTP ahora incluyen `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Content-Security-Policy` y omiten la versión del servidor.
3. **Validación de Rate Limiting:** Ráfagas de más de 30 peticiones en Backend y 15 en LDAP reciben de inmediato `429 Too Many Requests` con la cabecera `Retry-After`.
4. **Validación de Anonymous Bind:** Peticiones sin usuario o contraseña retornan `400 Bad Request` antes de interactuar con el directorio OpenLDAP.
