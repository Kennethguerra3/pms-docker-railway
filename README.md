# Plex Media Server - Railway Template

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/template)

Servidor multimedia completo con soporte para Google Drive ilimitado. Despliega en Railway en 5 minutos.

---

## 🚀 Inicio Rápido

1. Haz clic en el botón "Deploy on Railway" arriba
2. Obtén tu `PLEX_CLAIM` desde [plex.tv/claim](https://plex.tv/claim)
3. Configura las variables de entorno
4. ¡Listo! Accede a `https://tu-app.railway.app:32400/web`

---

## 📋 Tabla de Contenidos

- [Requisitos Previos](#-requisitos-previos)
- [Variables de Entorno](#%EF%B8%8F-variables-de-entorno)
- [Configuración de Google Drive (Service Account)](#-configuración-de-google-drive-service-account)
- [Configuración Post-Despliegue](#-configuración-post-despliegue)
- [Troubleshooting](#-troubleshooting)

---

## 📋 Requisitos Previos

1. **Cuenta de Plex**: [plex.tv](https://plex.tv)
2. **Claim Token**: [plex.tv/claim](https://plex.tv/claim) (válido 4 minutos)
3. **Cuenta de Railway**: [railway.app](https://railway.app)

---

## ⚙️ Variables de Entorno

### Variables Principales

| Variable | Descripción | Ejemplo | Requerido |
|----------|-------------|---------|-----------|
| `PLEX_CLAIM` | Token de reclamación (obtener en plex.tv/claim) | `claim-xxxxxxxxxxxx` | ✅ |
| `ADVERTISE_IP` | URL pública de Railway + puerto | `https://tu-app.railway.app:32400` | ✅ |
| `TZ` | Zona horaria | `America/Mexico_City` | ❌ |
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

## 💾 Volúmenes Persistentes

Railway monta automáticamente:

| Volumen | Ruta | Propósito |
|---------|------|-----------|
| `plex-config` | `/config` | **CRÍTICO**: Base de datos y configuración |
| `plex-data` | `/data` | Archivos multimedia (si no usas Google Drive) |
| `plex-transcode` | `/transcode` | Archivos temporales de transcodificación |

> ⚠️ **IMPORTANTE**: No elimines el volumen `/config` o perderás toda tu configuración.

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

## 🔧 Configuración Post-Despliegue

### 1. Obtener la URL Pública

Después del despliegue, Railway te asignará una URL pública:

```
https://<nombre-servicio>.up.railway.app
```

### 2. Configurar ADVERTISE_IP

Ve a las variables de entorno en Railway y actualiza:

```
ADVERTISE_IP=https://<nombre-servicio>.up.railway.app:32400
```

### 3. Acceder a Plex

Visita:

```
https://<nombre-servicio>.up.railway.app:32400/web
```

### 4. Configuración Inicial

1. Inicia sesión con tu cuenta de Plex
2. Configura tus bibliotecas apuntando a `/mnt/gdrive` (si usas Google Drive) o `/data`
3. Ajusta las configuraciones de transcodificación según tus necesidades

---

## 🌐 Configuración de Red

### Puerto Principal

- **Puerto**: `32400/TCP`
- **Protocolo**: HTTP/HTTPS
- **Uso**: Interfaz web y streaming

### Puertos Adicionales (Expuestos pero no públicos en Railway)

- `8324/TCP`: Roku via Plex Companion
- `32469/TCP`: Plex DLNA Server
- `1900/UDP`: Plex DLNA Server Discovery
- `32410-32414/UDP`: Network Discovery

> **Nota**: Railway solo expone el puerto 32400 públicamente. Los demás puertos están disponibles internamente.

---

## 🩺 Healthcheck

El servicio incluye un healthcheck automático:

- **Endpoint**: `http://localhost:32400/identity`
- **Intervalo**: Cada 5 segundos
- **Timeout**: 2 segundos
- **Reintentos**: 20 veces antes de marcar como unhealthy

---

## � Troubleshooting

### El servidor no es accesible externamente

- ✅ Verifica que `ADVERTISE_IP` esté configurado correctamente
- ✅ Asegúrate de que la URL incluya el puerto `:32400`
- ✅ Revisa los logs en Railway Dashboard

### El servidor se reinicia constantemente

- ✅ Verifica que el `PLEX_CLAIM` sea válido (no expirado)
- ✅ Revisa los logs para errores de permisos
- ✅ Asegúrate de que los volúmenes estén montados correctamente

### No puedo agregar bibliotecas

- ✅ Verifica que el volumen `/data` o `/mnt/gdrive` esté montado
- ✅ Asegúrate de tener archivos multimedia en la ruta correcta
- ✅ Revisa los permisos con `PLEX_UID` y `PLEX_GID`

### Google Drive: "Cannot read files"

- ✅ Verifica que compartiste la carpeta con el email de Service Account
- ✅ Asegúrate de dar permisos de **"Editor"**, no "Lector"
- ✅ Revisa logs: `[Rclone] ✓ Read access verified`

### Google Drive: "Invalid JSON"

- ✅ Abre el JSON con Bloc de notas (no Word)
- ✅ Copia TODO el contenido sin modificar
- ✅ Verifica que empieza con `{` y termina con `}`

### Problemas de transcodificación

- ✅ Railway tiene recursos limitados en el plan gratuito
- ✅ Considera actualizar a un plan con más CPU/RAM
- ✅ Ajusta la calidad de transcodificación en Plex

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
- [Repositorio GitHub Original](https://github.com/plexinc/pms-docker)
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
