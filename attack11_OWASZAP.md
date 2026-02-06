# Ataque 11: Escaneo de Vulnerabilidades Web con OWASP ZAP

## Objetivo

Utilizar OWASP ZAP (Zed Attack Proxy) para realizar un análisis automatizado de seguridad en un servidor web IIS, identificando vulnerabilidades comunes de aplicaciones web según el Top 10 de OWASP.

---

## Herramientas Utilizadas

- **Sistema Atacante:** Kali Linux (10.10.0.101)
- **Sistema Víctima:** Windows Server con IIS (10.10.0.20)
- **Herramienta:** OWASP ZAP (Zed Attack Proxy)
- **Red:** 10.10.0.0/24 (vmbr1 en Proxmox)

---

## ⚠️ Advertencia Legal

Este laboratorio es **únicamente con fines educativos** y debe realizarse en un entorno controlado.

- ❌ NO escanear sitios web sin autorización
- ❌ NO usar en entornos de producción sin permiso
- ✅ Solo en laboratorios virtualizados propios

**El escaneo no autorizado de aplicaciones web es ilegal.**

---

## Conceptos Previos

### ¿Qué es OWASP ZAP?

**OWASP ZAP** (Zed Attack Proxy) es una herramienta de seguridad de código abierto diseñada para encontrar vulnerabilidades en aplicaciones web.

- **Desarrollado por:** OWASP (Open Web Application Security Project)
- **Tipo:** Proxy interceptor y escáner de vulnerabilidades
- **Licencia:** Apache 2.0 (gratuito y open source)
- **Función:** Pruebas de penetración en aplicaciones web

### ¿Qué es IIS?

**IIS** (Internet Information Services) es el servidor web de Microsoft.

- **Desarrollado por:** Microsoft
- **Versiones:** IIS 7.0, 7.5, 8.0, 8.5, 10.0
- **Lenguajes soportados:** ASP.NET, PHP, Python, Node.js
- **Puerto por defecto:** 80 (HTTP), 443 (HTTPS)

### OWASP Top 10 (2021)

Las 10 vulnerabilidades más críticas en aplicaciones web:

1. **A01:2021** - Broken Access Control
2. **A02:2021** - Cryptographic Failures
3. **A03:2021** - Injection (SQL, XSS, etc.)
4. **A04:2021** - Insecure Design
5. **A05:2021** - Security Misconfiguration
6. **A06:2021** - Vulnerable and Outdated Components
7. **A07:2021** - Identification and Authentication Failures
8. **A08:2021** - Software and Data Integrity Failures
9. **A09:2021** - Security Logging and Monitoring Failures
10. **A10:2021** - Server-Side Request Forgery (SSRF)

---

## Parte 1: Preparación del Entorno

### Paso 1: Verificar conectividad con el servidor IIS
```bash
ping 10.10.0.20 -c 4
```

---

### Paso 2: Verificar que el servidor web está activo
```bash
curl -I http://10.10.0.20
```

**Alternativa con Nmap:**
```bash
nmap -p 80,443 10.10.0.20
```

---

### Paso 3: Identificar versión del servidor web
```bash
nmap -sV -p 80 10.10.0.20
```

---

### Paso 4: Verificar que OWASP ZAP está instalado
```bash
zaproxy --version
```

**Si no está instalado:**
```bash
sudo apt update
sudo apt install zaproxy -y
```

---

## Parte 2: Escaneo con OWASP ZAP (Modo GUI)

### Paso 1: Iniciar OWASP ZAP
```bash
zaproxy
```

O desde el menú de aplicaciones:
```
Applications > 03 - Web Application Analysis > zaproxy
```

---

### Paso 2: Configurar el objetivo

1. **En la ventana principal de ZAP:**
   - Ir a: `Quick Start` tab
   - En el campo **URL to attack:** ingresar `http://10.10.0.20`

2. **Seleccionar tipo de ataque:**
   - **Opción 1:** `Automated Scan` (escaneo automatizado completo)
   - **Opción 2:** `Manual Explore` (exploración manual)

Para este laboratorio usaremos **Automated Scan**.

---

### Paso 3: Iniciar el escaneo automatizado

1. Hacer clic en **Attack**
2. ZAP comenzará a:
   - 🕷️ **Spider** (rastrear el sitio web)
   - 🔍 **Active Scan** (escanear vulnerabilidades)

**Tiempo estimado:** 5-15 minutos (dependiendo del tamaño del sitio)

---

### Paso 4: Monitorear el progreso del escaneo

En la pestaña inferior verás:

- **Spider:** Progreso del rastreo de URLs
- **Active Scan:** Progreso del escaneo de vulnerabilidades
- **Alerts:** Vulnerabilidades encontradas en tiempo real

---

### Paso 5: Analizar los resultados

Una vez completado el escaneo, revisar la pestaña **Alerts**.

Las alertas se clasifican por nivel de riesgo:

| Color | Nivel de Riesgo | Icono |
|-------|-----------------|-------|
| 🔴 Rojo | **High** (Alto) | ⚠️ |
| 🟠 Naranja | **Medium** (Medio) | ⚠️ |
| 🟡 Amarillo | **Low** (Bajo) | ℹ️ |
| 🔵 Azul | **Informational** (Informativo) | ℹ️ |

---

## Parte 3: Escaneo con OWASP ZAP (Modo CLI)

### Paso 1: Escaneo básico desde terminal
```bash
zap-baseline.py -t http://10.10.0.20
```

**Parámetros:**
- `-t`: Target URL (objetivo)

---

### Paso 2: Escaneo completo con reporte
```bash
zap-full-scan.py -t http://10.10.0.20 -r reporte_zap.html
```

**Parámetros:**
- `-t`: Target URL
- `-r`: Nombre del archivo de reporte (HTML)

---

### Paso 3: Escaneo con autenticación

Si el sitio requiere autenticación:
```bash
zap-baseline.py -t http://10.10.0.20 \
  -z "-config api.key=your-api-key" \
  -r reporte_zap.html
```

---

### Paso 4: Visualizar el reporte generado
```bash
firefox reporte_zap.html &
```

---

### Paso 5: Generar reporte desde la GUI

1. En OWASP ZAP, ir a: **Report > Generate HTML Report**
2. Seleccionar ubicación: `/home/kali/reporte_zap_completo.html`
3. Hacer clic en **Save**

---

## Tabla Resumen del Escaneo

| Fase | Acción | Herramienta |
|------|--------|-------------|
| **Reconocimiento** | Verificar conectividad y puertos | Ping, Nmap |
| **Escaneo Spider** | Rastreo automático del sitio | OWASP ZAP |
| **Escaneo Activo** | Búsqueda de vulnerabilidades | OWASP ZAP |
| **Reporte** | Generación de reporte HTML | OWASP ZAP |

---

## Comandos Rápidos de Referencia

### Reconocimiento
```bash
ping 10.10.0.20 -c 4
curl -I http://10.10.0.20
nmap -sV -p 80,443 10.10.0.20
```

### Escaneo con ZAP (GUI)
```bash
zaproxy
# Luego: Quick Start > URL: http://10.10.0.20 > Attack
```

### Escaneo con ZAP (CLI)
```bash
# Escaneo básico
zap-baseline.py -t http://10.10.0.20

# Escaneo completo con reporte
zap-full-scan.py -t http://10.10.0.20 -r reporte_zap.html
```

### Ver reporte
```bash
firefox reporte_zap.html &
```
## Conclusiones

Este laboratorio demostró cómo:

- ✅ **Instalar y configurar** OWASP ZAP en Kali Linux
- ✅ **Realizar escaneos** automatizados de vulnerabilidades web
- ✅ **Identificar** vulnerabilidades comunes en servidores IIS
- ✅ **Generar reportes** profesionales de seguridad
- ✅ **Utilizar** tanto la interfaz gráfica como la línea de comandos
### Tipos de Vulnerabilidades Detectadas por ZAP

- **Security Headers Faltantes** (Missing Security Headers)
- **Directory Browsing** (Listado de directorios)
- **Cookies Inseguras** (Cookie Without Secure Flag)
- **Information Disclosure** (Exposición de información del servidor)
- **SQL Injection** (Inyección SQL)
- **Cross-Site Scripting (XSS)**
- **CSRF** (Cross-Site Request Forgery)

---