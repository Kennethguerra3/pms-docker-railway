# Plex Media Server - Docker & Railway

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/template)

Servidor multimedia completo con soporte para Google Drive ilimitado. Despliega en Railway en 5 minutos o ejecuta localmente con Docker.

---

## 🚀 Inicio Rápido

### Opción 1: Desplegar en Railway (Recomendado)

1. Haz clic en el botón "Deploy on Railway" arriba
2. Obtén tu `PLEX_CLAIM` desde [plex.tv/claim](https://plex.tv/claim)
3. Configura las variables de entorno
4. ¡Listo! Accede a `https://tu-app.railway.app:32400/web`

### Opción 2: Docker Local

```bash
docker run \
-d \
--name plex \
--network=host \
-e TZ="America/New_York" \
-e PLEX_CLAIM="<tu-claim-token>" \
-v <ruta/config>:/config \
-v <ruta/transcode>:/transcode \
-v <ruta/media>:/data \
plexinc/pms-docker
```

---

## 📋 Tabla de Contenidos

- [Despliegue en Railway](#-despliegue-en-railway)
- [Configuración de Google Drive (Service Account)](#-configuración-de-google-drive-service-account)
- [Variables de Entorno](#%EF%B8%8F-variables-de-entorno)
- [Docker Local](#-uso-con-docker-local)
- [Troubleshooting](#-troubleshooting)

---

## 🌐 Despliegue en Railway

### Requisitos Previos

1. **Cuenta de Plex**: [plex.tv](https://plex.tv)
2. **Claim Token**: [plex.tv/claim](https://plex.tv/claim) (válido 4 minutos)
3. **Cuenta de Railway**: [railway.app](https://railway.app)

### Variables de Entorno Requeridas

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `PLEX_CLAIM` | Token de reclamación (obtener en plex.tv/claim) | `claim-xxxxxxxxxxxx` |
| `ADVERTISE_IP` | URL pública de Railway + puerto | `https://tu-app.railway.app:32400` |
| `TZ` | Zona horaria | `America/Mexico_City` |

### Volúmenes Persistentes

Railway monta automáticamente:

| Volumen | Ruta | Propósito |
|---------|------|-----------|
| `plex-config` | `/config` | **CRÍTICO**: Base de datos y configuración |
| `plex-data` | `/data` | Archivos multimedia |
| `plex-transcode` | `/transcode` | Archivos temporales |

> ⚠️ **IMPORTANTE**: No elimines el volumen `/config` o perderás toda tu configuración.

### Configuración Post-Despliegue

1. **Obtener URL pública** de Railway Dashboard
2. **Actualizar** `ADVERTISE_IP=https://tu-app.railway.app:32400`
3. **Acceder** a `https://tu-app.railway.app:32400/web`
4. **Configurar bibliotecas** apuntando a `/data` o `/mnt/gdrive`

---

## 📁 Configuración de Google Drive (Service Account)

### ⭐ Método Recomendado: Service Account (5 minutos)

✅ **Sin instalar nada en tu PC**  
✅ **Solo copiar/pegar un archivo JSON**  
✅ **Nunca expira**  
✅ **Almacenamiento ilimitado** (según tu plan de Google)

### Paso 1: Crear Proyecto en Google Cloud Console

1. Ve a [console.cloud.google.com](https://console.cloud.google.com)
2. Clic en selector de proyectos → **"Nuevo Proyecto"**
3. Nombre: `Plex Media Server`
4. Clic en **"Crear"**

### Paso 2: Habilitar Google Drive API

1. Menú (☰) → **"APIs y servicios"** → **"Biblioteca"**
2. Busca: `Google Drive API`
3. Clic en **"Habilitar"**

### Paso 3: Crear Service Account

1. Menú (☰) → **"IAM y administración"** → **"Cuentas de servicio"**
2. Clic en **"+ Crear cuenta de servicio"**
3. Nombre: `plex-gdrive`
4. Clic en **"Crear y continuar"**
5. **Omitir** roles → **"Continuar"** → **"Listo"**

### Paso 4: Descargar Credenciales JSON

1. Clic en el email de la Service Account (`plex-gdrive@...`)
2. Pestaña **"Claves"** → **"Agregar clave"** → **"Crear clave nueva"**
3. Tipo: **JSON** → **"Crear"**
4. Se descarga automáticamente un archivo `.json`

**⚠️ IMPORTANTE:** Copia el email de la Service Account (lo necesitarás en el siguiente paso):
```
plex-gdrive@tu-proyecto-123456.iam.gserviceaccount.com
```

### Paso 5: Compartir Carpeta de Google Drive

1. Ve a [drive.google.com](https://drive.google.com)
2. Crea carpeta: **"Plex"**
3. Dentro de `Plex`, crea subcarpetas:
   - `Movies`
   - `TV Shows`
   - `Music`
4. **Clic derecho** en carpeta `Plex` → **"Compartir"**
5. Pega el email de la Service Account
6. Cambia permiso a **"Editor"**
7. **Desactiva** "Notificar a las personas"
8. Clic en **"Compartir"**

### Paso 6: Configurar en Railway

1. Abre el archivo `.json` descargado con **Bloc de notas**
2. **Selecciona TODO** (Ctrl+A) y **copia** (Ctrl+C)
3. En Railway Dashboard → **"Variables"**:

| Variable | Valor |
|----------|-------|
| `ENABLE_RCLONE` | `true` |
| `RCLONE_SERVICE_ACCOUNT_JSON` | *Pegar todo el contenido del JSON* |
| `RCLONE_REMOTE_NAME` | `gdrive` |
| `RCLONE_REMOTE_PATH` | `/Plex` |

1. Clic en **"Deploy"**

### Paso 7: Subir Películas

Organiza tus archivos en Google Drive:

```
Google Drive/
└── Plex/
    ├── Movies/
    │   ├── Avatar (2009)/
    │   │   └── Avatar (2009).mkv
    │   └── Inception (2010)/
    │       └── Inception (2010).mp4
    └── TV Shows/
        └── Breaking Bad/
            └── Season 01/
                ├── Breaking Bad - S01E01.mkv
                └── Breaking Bad - S01E02.mkv
```

**Métodos para subir:**

- 🌐 Navegador: [drive.google.com](https://drive.google.com)
- 💻 App de escritorio: Google Drive para PC
- 📱 App móvil: Google Drive

### Paso 8: Configurar Bibliotecas en Plex

1. Accede a Plex: `https://tu-app.railway.app:32400/web`
2. Clic en **"+"** junto a "Bibliotecas"
3. Selecciona tipo: **"Películas"**
4. Clic en **"Examinar carpeta"**
5. Navega a: `/mnt/gdrive/Plex/Movies`
6. Clic en **"Agregar biblioteca"**

### ✅ Verificación

Revisa los logs en Railway:

```
[Rclone] Using Service Account authentication (recommended)
[Rclone] ✓ Service Account configuration created
[Rclone] ✓ Google Drive mounted successfully at /mnt/gdrive
[Rclone] ✓ Read access verified
```

### 📖 Guía Detallada

Para troubleshooting y configuración avanzada, consulta:

- **Service Account**: [SERVICE_ACCOUNT_SETUP.md](SERVICE_ACCOUNT_SETUP.md)
- **OAuth (Avanzado)**: [GOOGLE_DRIVE_SETUP.md](GOOGLE_DRIVE_SETUP.md)

---

## ⚙️ Variables de Entorno

### Variables Principales

| Variable | Descripción | Default | Requerido |
|----------|-------------|---------|-----------|
| `PLEX_CLAIM` | Token de reclamación de plex.tv/claim | - | ✅ |
| `TZ` | Zona horaria (ej: `America/New_York`) | `UTC` | ❌ |
| `ADVERTISE_IP` | URL pública (Railway) | - | ✅ (Railway) |
| `PLEX_UID` | User ID para permisos | `1000` | ❌ |
| `PLEX_GID` | Group ID para permisos | `1000` | ❌ |

### Variables de Google Drive (Service Account)

| Variable | Descripción | Default | Requerido |
|----------|-------------|---------|-----------|
| `ENABLE_RCLONE` | Habilitar montaje de Google Drive | `false` | ❌ |
| `RCLONE_SERVICE_ACCOUNT_JSON` | JSON completo de Service Account | - | ❌ |
| `RCLONE_REMOTE_NAME` | Nombre del remote | `gdrive` | ❌ |
| `RCLONE_REMOTE_PATH` | Ruta en Google Drive | `/` | ❌ |

### Variables Avanzadas

| Variable | Descripción | Default |
|----------|-------------|---------|
| `ALLOWED_NETWORKS` | Redes permitidas sin autenticación | - |
| `CHANGE_CONFIG_DIR_OWNERSHIP` | Cambiar permisos de `/config` | `true` |
| `RCLONE_CONFIG` | Config OAuth en base64 (avanzado) | - |

---

## 🐳 Uso con Docker Local

### Networking: Host (Recomendado)

```bash
docker run \
-d \
--name plex \
--network=host \
-e TZ="America/New_York" \
-e PLEX_CLAIM="<claim-token>" \
-v /ruta/config:/config \
-v /ruta/transcode:/transcode \
-v /ruta/media:/data \
plexinc/pms-docker
```

### Networking: Bridge

```bash
docker run \
-d \
--name plex \
-p 32400:32400/tcp \
-p 8324:8324/tcp \
-p 32469:32469/tcp \
-p 1900:1900/udp \
-p 32410:32410/udp \
-p 32412:32412/udp \
-p 32413:32413/udp \
-p 32414:32414/udp \
-e TZ="America/New_York" \
-e PLEX_CLAIM="<claim-token>" \
-e ADVERTISE_IP="http://<tu-ip>:32400/" \
-v /ruta/config:/config \
-v /ruta/transcode:/transcode \
-v /ruta/media:/data \
plexinc/pms-docker
```

### Docker Compose

```yaml
version: '3.8'
services:
  plex:
    image: plexinc/pms-docker
    container_name: plex
    network_mode: host
    environment:
      - TZ=America/New_York
      - PLEX_CLAIM=<claim-token>
    volumes:
      - ./config:/config
      - ./transcode:/transcode
      - ./media:/data
    restart: unless-stopped
```

### Comandos Útiles

```bash
# Iniciar contenedor
docker start plex

# Detener contenedor
docker stop plex

# Ver logs
docker logs -f plex

# Acceso shell
docker exec -it plex /bin/bash

# Reiniciar y actualizar
docker restart plex
```

---

## 🔧 Configuración Avanzada

### Intel Quick Sync (Hardware Transcoding)

Requiere Plex Pass y CPU Intel con Quick Sync:

```bash
docker run \
-d \
--name plex \
--network=host \
--device=/dev/dri:/dev/dri \
-e TZ="America/New_York" \
-e PLEX_CLAIM="<claim-token>" \
-v /ruta/config:/config \
-v /ruta/transcode:/transcode \
-v /ruta/media:/data \
plexinc/pms-docker
```

Luego en Plex Web:

1. Settings → Server → Transcoder
2. Activar "Show Advanced"
3. Activar "Use hardware acceleration when available"

### Permisos de Usuario

Para que Plex use tus permisos de usuario:

```bash
# Obtener tu UID/GID
id $(whoami)
# Salida: uid=1001(usuario) gid=1001(usuario)

# Usar en docker run
-e PLEX_UID=1001 \
-e PLEX_GID=1001
```

---

## 🐛 Troubleshooting

### Railway: Servidor no accesible externamente

- ✅ Verifica `ADVERTISE_IP=https://tu-app.railway.app:32400`
- ✅ Asegúrate de incluir el puerto `:32400`
- ✅ Revisa logs en Railway Dashboard

### Google Drive: "Cannot read files"

- ✅ Verifica que compartiste la carpeta con el email de Service Account
- ✅ Asegúrate de dar permisos de **"Editor"**, no "Lector"
- ✅ Revisa logs: `[Rclone] ✓ Read access verified`

### Google Drive: "Invalid JSON"

- ✅ Abre el JSON con Bloc de notas (no Word)
- ✅ Copia TODO el contenido sin modificar
- ✅ Verifica que empieza con `{` y termina con `}`

### Docker: Servidor se reinicia constantemente

- ✅ Verifica que `PLEX_CLAIM` no haya expirado (4 minutos)
- ✅ Revisa logs: `docker logs plex`
- ✅ Verifica permisos de volúmenes

### Docker: No puedo agregar bibliotecas

- ✅ Verifica que el volumen `/data` esté montado
- ✅ Asegúrate de tener archivos en `/data`
- ✅ Revisa permisos con `PLEX_UID` y `PLEX_GID`

### Headless Server (Sin GUI)

Si no agregaste `PLEX_CLAIM` al inicio, usa SSH tunneling:

```bash
ssh usuario@ip-servidor -L 32400:localhost:32400 -N
```

Luego accede a `http://localhost:32400/web` en tu PC.

---

## 📊 Comparación: Service Account vs OAuth

| Característica | Service Account | OAuth Personal |
|----------------|-----------------|----------------|
| **Instalación en PC** | ❌ No requiere | ✅ Requiere Rclone |
| **Complejidad** | ⭐ Muy fácil | ⭐⭐⭐ Media |
| **Tiempo setup** | ~5 minutos | ~15 minutos |
| **Expiración** | ♾️ Nunca | ⚠️ Puede expirar |
| **Seguridad** | ✅ Acceso limitado | ⚠️ Acceso total |
| **Recomendado para** | Todos | Usuarios avanzados |

---

## 📚 Recursos Adicionales

- [Documentación Oficial de Plex](https://support.plex.tv/)
- [Repositorio GitHub](https://github.com/plexinc/pms-docker)
- [Documentación de Railway](https://docs.railway.app/)
- [Foro de la Comunidad Plex](https://forums.plex.tv/)
- [Guía Service Account Detallada](SERVICE_ACCOUNT_SETUP.md)
- [Guía OAuth Avanzada](GOOGLE_DRIVE_SETUP.md)

---

## 📄 Licencia

Este proyecto usa el contenedor oficial de Plex Media Server. Consulta la [licencia de Plex](https://www.plex.tv/about/privacy-legal/plex-terms-of-service/).

---

## 🤝 Contribuciones

Si encuentras problemas o tienes sugerencias, abre un issue en [plexinc/pms-docker](https://github.com/plexinc/pms-docker/issues).
