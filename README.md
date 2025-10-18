# Evaluación Sumativa – Laboratorio 2: Implementación de Servicios en Nube Privada (Docker)

**Asignatura:** Computación en la Nube – IEI-075  
**Unidad:** II – Implementación de servicios en una nube privada  
**Tipo:** Laboratorio práctico   
**Ponderación:** 40% del proceso práctico  

---

## Enunciado 

Debes **dockerizar una aplicación PHP con base de datos MariaDB**, implementando una infraestructura funcional y segura compuesta por dos servicios interconectados:

1. Un contenedor **web** que ejecute un sitio PHP entregado (archivo `index.php` con monitor automático).  
2. Un contenedor **de base de datos** que cargue y administre la base `empresa` con una tabla inicial `clientes` a partir del archivo `init.sql`.

La aplicación web verifica automáticamente la configuración del entorno, mostrando los resultados de conexión y cumplimiento mediante un **monitor de validaciones**.  
Tu objetivo es lograr que **todos los indicadores del monitor aparezcan en “OK”**, cumpliendo las especificaciones técnicas establecidas.

---

## Requerimientos Técnicos

1. **Contenedores obligatorios**
   - `nube-web`: PHP 8.2 + Apache  
   - `nube-db`: MariaDB 11.x  
2. **Red compartida:** `nube-net`  
3. **Puerto de acceso:** `8080` (host → contenedor web)  
4. **Persistencia:** volumen `db_data` para la base de datos.  
5. **Configuración por variables de entorno:**
   - `DB_HOST=nube-db`  
   - `DB_PORT=3306`  
   - `DB_NAME=empresa`  
   - `DB_USER=appuser`  
   - `DB_PASSWORD=apppass`  
   - `APP_REQUIRED_PORT=8080`  
6. **Archivo `init.sql`:** inicializa la base `empresa` con tabla `clientes` y 3 registros.  
7. **Seguridad:** el contenedor de BD **no debe exponer puertos al host**.  
8. **Requisitos de operación:**
   - Healthcheck en ambos servicios.  
   - Reinicio automático (`restart: unless-stopped`).  
   - Uso de imágenes con versión explícita (no `latest`).  

---

## Actividad de Consola (Administración Real de BD)

Accede al contenedor de base de datos mediante consola y crea una **nueva tabla** (por ejemplo `cursos`) dentro de la base `empresa`.  
Agrega **al menos dos registros** en ella utilizando comandos SQL.  
La existencia de esta tabla y sus registros será verificada automáticamente en el monitor del sitio web bajo los indicadores:
- “Cantidad de tablas”
- “Nombres de tablas”

---

## Estructura del Proyecto

```bash
eval2/
├─ docker-compose.yml
├─ init.sql
├─ app-php/
│  ├─ Dockerfile
│  └─ index.php
└─ README.md
```

---

## Evidencias de Entrega

- Proyecto completo con estructura exacta.  
- Captura del monitor web mostrando todos los indicadores en **OK**, incluyendo la nueva tabla creada.  
- Salida de `docker compose ps`.  
- Fragmento de logs de salud (`healthcheck`) de la base de datos.  
- Comandos utilizados para crear e insertar en la nueva tabla.  
- Archivo `README.md` con instrucciones reproducibles (inicio, detención, backup, restore).

---

## Rúbrica de Evaluación (100 puntos)

| **Criterio** | **Descripción** | **Puntaje Máx.** |
|---------------|----------------|:----------------:|
| **A. Estructura de proyecto y Compose válido** | | **10 pts** |
| Árbol exacto del proyecto (`eval2/`, `docker-compose.yml`, `init.sql`, `app-php/Dockerfile`, `app-php/index.php`, `README.md`) | 5 |
| `docker-compose.yml` correcto, levanta sin errores (`docker compose up -d`) | 5 |
| **B. Nombres, red y puerto exigidos** | | **15 pts** |
| Contenedor web llamado y hosteado como `nube-web` | 5 |
| Contenedor de base de datos llamado `nube-db` | 3 |
| Red única `nube-net` compartida por ambos servicios | 3 |
| Publicación `8080:80` y variable `APP_REQUIRED_PORT=8080` | 4 |
| **C. Seguridad de red** | | **8 pts** |
| Servicio `db` no expone puertos al host (solo accesible por red interna) | 8 |
| **D. Persistencia y reinicios** | | **10 pts** |
| Volumen `db_data` correctamente montado | 5 |
| Persistencia verificada tras reinicio sin eliminar volúmenes | 5 |
| **E. Operación y hardening de contenedores** | | **12 pts** |
| `healthcheck` funcional en `db` y dependencia en `web` | 4 |
| `healthcheck` HTTP en `web` (curl localhost) | 4 |
| Política `restart: unless-stopped` configurada en ambos | 4 |
| **F. Inicialización de BD por `init.sql`** | | **10 pts** |
| `init.sql` montado en `/docker-entrypoint-initdb.d/init.sql:ro` | 4 |
| Base `empresa`, tabla `clientes` creada con 3 registros iniciales | 6 |
| **G. Tarea de consola (administración real de BD)** | | **15 pts** |
| Acceso a consola del contenedor DB y creación de nueva tabla (ej. `cursos`) | 7 |
| Inserción de ≥2 registros en la nueva tabla | 8 |
| **H. Monitor web (verificación automática)** | | **15 pts** |
| “Nombre contenedor web” = OK | 3 |
| “Puerto de acceso (host)” = OK | 3 |
| “DB_HOST” = OK | 3 |
| “Conexión a BD” y “Total filas en clientes” correctos | 3 |
| “Cantidad de tablas” ≥ 2 y listado de nombres actualizado | 3 |
| **I. Documentación y reproducibilidad** | | **5 pts** |
| `README.md` con comandos (`up`, `down`, `logs`, `exec`, backup/restore`, verificación`) | 5 |
| **Total** |  | **100 pts** |

---

### **Descuentos automáticos**

| Condición | Descuento |
|------------|-----------|
| Puerto de BD expuesto al host | −8 pts |
| Imágenes sin versión fija (`latest`) | −4 pts |
| Contenedor web ejecutado como root | −4 pts |
| Manipular `index.php` para forzar “OK” | −10 pts |
| Sin volumen persistente | −10 pts |
| No ejecutar la tarea de consola (crear tabla + inserts) | −15 pts |


# Comandos esenciales – Laboratorio 2: Dockerización de Servicios

## Construcción y gestión de contenedores

```bash
# Construir e iniciar todos los contenedores en segundo plano
docker compose up -d --build

# Verificar el estado de los servicios activos
docker compose ps

# Ver los logs de un servicio específico (ejemplo: base de datos)
docker compose logs -f <nombre_servicio>

# Detener los contenedores sin eliminar volúmenes
docker compose down

# Detener y eliminar contenedores + volúmenes (reinicia todo el entorno)
docker compose down -v
```

---

## Acceso a consolas internas

```bash
# Ingresar a la terminal del contenedor web
docker exec -it <nombre_contenedor_web> bash

# Ingresar a la base de datos dentro del contenedor
docker exec -it <nombre_contenedor_db> mariadb -u<usuario> -p<contraseña> -D <nombre_bd>
```

---

## Definir hostname en un servicio (docker-compose.yml)

```yaml
services:
  web:
    build: ./<carpeta_app>
    container_name: <nombre_contenedor_web>
    hostname: <nombre_host_web>   # Define el hostname visible dentro del contenedor
```

---

## Cargar automáticamente un archivo SQL al crear la base de datos

```yaml
services:
  db:
    image: mariadb:<versión>
    container_name: <nombre_contenedor_db>
    environment:
      MARIADB_DATABASE: <nombre_bd>
      MARIADB_USER: <usuario>
      MARIADB_PASSWORD: <contraseña>
      MARIADB_ROOT_PASSWORD: <contraseña_root>
    volumes:
      - <nombre_volumen>:/var/lib/mysql
      - ./<archivo_sql>:/docker-entrypoint-initdb.d/<archivo_sql>:ro
```

> Los archivos `.sql` o `.sh` montados en `/docker-entrypoint-initdb.d/` se ejecutan automáticamente al crear la base por primera vez.

---

## Reiniciar completamente el entorno (forzar ejecución del init.sql)

```bash
docker compose down -v
docker compose up -d
```

