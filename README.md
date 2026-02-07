# Plex Media Server - Railway Template

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/template)

Servidor multimedia completo con soporte para Google Drive ilimitado. Despliega en Railway en 5 minutos.

---

## 🚀 Inicio Rápido

1. Haz clic en el botón "Deploy on Railway" arriba
2. Obtén tu `PLEX_CLAIM` desde [plex.tv/claim](https://plex.tv/claim)
3. Configura las variables de entorno
4. Espera a que termine el deploy
5. Configura TCP Proxy en Settings → Networking
6. Agrega `ADVERTISE_IP` con la URL del TCP Proxy
7. ¡Listo! Accede desde [app.plex.tv](https://app.plex.tv)

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
| `PLEX_CLAIM` | Token de reclamación | `claim-xxxxxxxxxxxx` | ✅ |
| `TZ` | Zona horaria | `America/Mexico_City` | ❌ |
| `ADVERTISE_IP` | URL TCP Proxy de Railway (configurar después del deploy) | `tcp://monorail.proxy.rlwy.net:12345` | ⚠️ Post-deploy |

### Variables de Google Drive (Service Account)

| Variable | Descripción | Default |
|----------|-------------|---------|
| `ENABLE_RCLONE` | Habilitar montaje de Google Drive | `false` |
| `RCLONE_SERVICE_ACCOUNT_JSON` | JSON completo de Service Account | - |
| `RCLONE_REMOTE_NAME` | Nombre del remote | `gdrive` |
| `RCLONE_REMOTE_PATH` | Ruta en Google Drive | `/` |

---

## 💾 Volúmenes Persistentes

Railway monta automáticamente:

| Volumen | Ruta | Propósito |
|---------|------|-----------|
| `plex-config` | `/config` | Base de datos y configuración |
| `plex-data` | `/data` | Archivos multimedia |
| `plex-transcode` | `/transcode` | Archivos temporales |

> ⚠️ **IMPORTANTE**: No elimines el volumen `/config` o perderás toda tu configuración.

---

## 📁 Configuración de Google Drive (Service Account)

### ⭐ Método Recomendado: 5 minutos

✅ Sin instalar nada en tu PC  
✅ Solo copiar/pegar un archivo JSON  
✅ Nunca expira  

### Paso 1: Crear Proyecto en Google Cloud

1. Ve a [console.cloud.google.com](https://console.cloud.google.com)
2. Clic en "Nuevo Proyecto"
3. Nombre: `Plex Media Server`
4. Clic en "Crear"

### Paso 2: Habilitar Google Drive API

1. Menú → "APIs y servicios" → "Biblioteca"
2. Busca: `Google Drive API`
3. Clic en "Habilitar"

### Paso 3: Crear Service Account

1. Menú → "IAM y administración" → "Cuentas de servicio"
2. Clic en "+ Crear cuenta de servicio"
3. Nombre: `plex-gdrive`
4. Clic en "Crear y continuar" → "Listo"

### Paso 4: Descargar Credenciales JSON

1. Clic en el email de la Service Account
2. Pestaña "Claves" → "Agregar clave" → "Crear clave nueva"
3. Tipo: **JSON** → "Crear"
4. Copia el email de la Service Account:

```
plex-gdrive@tu-proyecto-123456.iam.gserviceaccount.com
```

### Paso 5: Compartir Carpeta de Google Drive

1. Ve a [drive.google.com](https://drive.google.com)
2. Crea carpeta: **"Plex"**
3. Dentro crea: `Movies`, `TV Shows`, `Music`
4. Clic derecho en "Plex" → "Compartir"
5. Pega el email de la Service Account
6. Cambia permiso a **"Editor"**
7. Desactiva "Notificar a las personas"
8. Clic en "Compartir"

### Paso 6: Configurar en Railway

1. Abre el archivo `.json` con Bloc de notas
2. Copia TODO el contenido (Ctrl+A, Ctrl+C)
3. En Railway → "Variables":

| Variable | Valor |
|----------|-------|
| `ENABLE_RCLONE` | `true` |
| `RCLONE_SERVICE_ACCOUNT_JSON` | *Pegar el JSON completo* |
| `RCLONE_REMOTE_NAME` | `gdrive` |
| `RCLONE_REMOTE_PATH` | `/Plex` |

### Paso 7: Subir Películas

Organiza tus archivos:

```
Google Drive/Plex/
├── Movies/
│   ├── Avatar (2009)/
│   │   └── Avatar (2009).mkv
│   └── Inception (2010)/
│       └── Inception (2010).mp4
└── TV Shows/
    └── Breaking Bad/
        └── Season 01/
            └── Breaking Bad - S01E01.mkv
```

### Paso 8: Configurar Bibliotecas en Plex

1. Accede a `https://tu-app.railway.app:32400/web`
2. Clic en "+" junto a "Bibliotecas"
3. Selecciona tipo: "Películas"
4. Navega a: `/mnt/gdrive/Plex/Movies`
5. Clic en "Agregar biblioteca"

### ✅ Verificación

Revisa los logs en Railway:
```
[Rclone] Using Service Account authentication
[Rclone] ✓ Google Drive mounted successfully
[Rclone] ✓ Read access verified
```

**Guía detallada**: [SERVICE_ACCOUNT_SETUP.md](SERVICE_ACCOUNT_SETUP.md)

---

## 🔧 Configuración Post-Despliegue

### 1. Exponer Puerto con TCP Proxy

Después del deploy, Railway NO genera una URL automáticamente. Debes configurar el networking:

1. Ve a tu servicio en Railway Dashboard
2. Pestaña **"Settings"** → **"Networking"**
3. Clic en **"Add TCP Proxy"**
4. Puerto: `32400`
5. Railway generará una URL tipo:

   ```
   tcp://monorail.proxy.rlwy.net:12345
   ```

### 2. Configurar ADVERTISE_IP

Copia la URL del TCP Proxy y agrégala como variable de entorno:

1. Pestaña **"Variables"**
2. Agrega nueva variable:

   ```
   ADVERTISE_IP=tcp://monorail.proxy.rlwy.net:12345
   ```

3. El servicio se reiniciará automáticamente

### 3. Acceder a Plex

Usa la URL del TCP Proxy para acceder:
```
tcp://monorail.proxy.rlwy.net:12345/web
```

O accede directamente desde [app.plex.tv](https://app.plex.tv) - Plex detectará tu servidor automáticamente.

---

## 🐛 Troubleshooting

### El servidor no es accesible

- ✅ Verifica `ADVERTISE_IP` con el puerto `:32400`
- ✅ Revisa los logs en Railway Dashboard

### El servidor se reinicia constantemente

- ✅ Verifica que `PLEX_CLAIM` no esté expirado
- ✅ Revisa los logs para errores

### Google Drive: "Cannot read files"

- ✅ Verifica que compartiste la carpeta con el email de Service Account
- ✅ Asegúrate de dar permisos de "Editor"
- ✅ Revisa logs: `[Rclone] ✓ Read access verified`

### Google Drive: "Invalid JSON"

- ✅ Abre el JSON con Bloc de notas (no Word)
- ✅ Copia TODO sin modificar
- ✅ Verifica que empieza con `{` y termina con `}`

---

## � Comparación de Métodos

| Característica | Service Account | OAuth |
|----------------|-----------------|-------|
| Instalación en PC | ❌ No requiere | ✅ Requiere Rclone |
| Complejidad | ⭐ Muy fácil | ⭐⭐⭐ Media |
| Tiempo setup | ~5 minutos | ~15 minutos |
| Expiración | ♾️ Nunca | ⚠️ Puede expirar |
| Recomendado | ✅ Todos | Usuarios avanzados |

---

## 📚 Recursos

- [Documentación Oficial de Plex](https://support.plex.tv/)
- [Repositorio Original](https://github.com/plexinc/pms-docker)
- [Documentación de Railway](https://docs.railway.app/)
- [Guía Service Account Detallada](SERVICE_ACCOUNT_SETUP.md)
- [Guía OAuth Avanzada](GOOGLE_DRIVE_SETUP.md)

---

## 📄 Licencia

Este proyecto usa el contenedor oficial de Plex Media Server. Consulta la [licencia de Plex](https://www.plex.tv/about/privacy-legal/plex-terms-of-service/).
