# 🕵️‍♂️ Laboratorio: ARP Spoofing & Man-in-the-Middle (MitM)

> **DISCLAIMER:** Este laboratorio tiene fines **estrictamente educativos** para la asignatura de Seguridad en Redes. La interceptación de tráfico en redes ajenas sin consentimiento explícito es ilegal. Las pruebas se realizaron en un entorno de laboratorio virtual controlado y aislado.

## 🎯 Objetivos del Laboratorio
1.  **Comprender la vulnerabilidad del protocolo ARP:** Entender por qué la falta de autenticación en ARP permite la suplantación de identidad.
2.  **Configurar un entorno de ataque:** Preparar máquinas virtuales en modo "Puente" (Bridge) para interactuar con la red física.
3.  **Ejecutar un ataque MitM:** Utilizar **Ettercap** para interceptar la comunicación entre una víctima y el Router.
4.  **Capturar credenciales:** Obtener usuarios y contraseñas transmitidos en texto plano (HTTP) mediante *sniffing*.

---

## 🛠️ Escenario y Requisitos

### Topología
* **Atacante:** Kali Linux.
    * Herramienta: `Ettercap (Graphical)`.
* **Víctima:** Windows 10/11 (Máquina Física o VM).
* **Gateway:** Router doméstico (Puerta de enlace).

### ⚙️ Configuración Crítica de Red (Bridge Mode)
Para que el ataque funcione, la máquina atacante **NO puede estar en NAT**.
* **Problema del NAT:** Aísla al atacante en una subred virtual (`172.x.x.x`), impidiendo el envenenamiento ARP de la red real.
* **Solución:** Se configuro en modo **Bridge**.
* **Resultado:** Kali obtuvo una IP del rango local (`192.168.100.x`), logrando visibilidad directa de la víctima.

---

## 🧠 Marco Teórico: ¿Qué es el ARP Spoofing?

El protocolo **ARP** (Address Resolution Protocol) mapea direcciones IP (Capa 3) a direcciones MAC (Capa 2).

El ataque se basa en dos debilidades:
1.  **Falta de Verificación:** Los dispositivos confían ciegamente en cualquier respuesta ARP que reciben.
2.  **Gratuitous ARP:** Un dispositivo puede anunciar "Yo tengo esta IP" sin que nadie se lo haya preguntado.

**Mecánica del Ataque (Man-in-the-Middle):**
El atacante envía mensajes ARP falsos constantemente:
* Le dice a la **Víctima**: *"Yo soy el Router"*.
* Le dice al **Router**: *"Yo soy la Víctima"*.
Esto fuerza a que todo el tráfico pase a través del atacante antes de llegar a su destino real.

---

## 🚀 Ejecución Paso a Paso

### 1. Reconocimiento
Identificamos los actores en la red.
* **En Windows (Víctima):**
    ```cmd
    ipconfig
    arp -a  (Tomar nota de la MAC real del Router)
    ```
* **En Kali (Atacante):**
    ```bash
    sudo ettercap -G
    ```

### 2. Configuración de Objetivos (Targeting)
Para lograr una interceptación bidireccional, configuramos:

* **Target 1 (Víctima):** IP de Windows.
    * *Objetivo:* Capturar tráfico de subida (Upload/Peticiones).
* **Target 2 (Router/Gateway):** IP de la Puerta de Enlace (`192.168.100.1`).
    * *Objetivo:* Capturar tráfico de bajada (Download/Respuestas).

### 3. Envenenamiento (Poisoning)
* **Acción:** Menú `MITM` > `ARP Poisoning` > ☑️ `Sniff remote connections`.
* **Resultado en consola:**
    ```text
    ARP poisoning victims:
    GROUP 1 : 192.168.X.X 
    GROUP 2 : 192.168.X.X
    ```

### 4. Validación del Ataque
En la máquina víctima, al ejecutar `arp -a` nuevamente, se observa que la **dirección MAC del Router ha cambiado** y ahora es idéntica a la dirección MAC de Kali Linux. La tabla ARP ha sido envenenada exitosamente.

---

## 📸 Resultados:

Se realizó una prueba de concepto accediendo a un sitio web vulnerable (HTTP) desde la víctima.

* **URL:** `http://testphp.vulnweb.com/login.php`
* **Credenciales ingresadas:** `admin` / `supersecreto123`

**Evidencia en Ettercap:**
La consola registró la captura de las credenciales en texto plano:

text
``
HTTP : 192.168.100.X -> USER: admin  PASS: supersecreto123  INFO: [http://testphp.vulnweb.com/userinfo.php](http://testphp.vulnweb.com/userinfo.php)
``

## Medidas de Mitigación
Para defenderse de este ataque en un entorno empresarial (Cisco), se recomienda:

1. Dynamic ARP Inspection (DAI): Característica de seguridad que valida los paquetes ARP en la red. Intercepta, registra y descarta paquetes ARP con asociaciones de direcciones MAC a IP no válidas.
2. DHCP Snooping: Crea una base de datos de confianza (Binding Database) que DAI utiliza para saber qué MAC corresponde legítimamente a qué IP.
3. Cifrado (HTTPS/VPN): Aunque no evita el MitM, hace que los datos capturados sean ilegibles.
