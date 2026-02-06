# 🌊 Laboratorio: Ataque de Inundación MAC y Defensa (Port Security)

> **DISCLAIMER:** Este laboratorio tiene fines **estrictamente educativos**. La ejecución de herramientas de inundación como `macof` en redes de producción puede causar denegación de servicio (DoS) y caída de la red. Realice estas pruebas únicamente en entornos controlados y autorizados.

## 🎯 Objetivos del Laboratorio
1.  **Comprender la vulnerabilidad:** Entender cómo funciona la tabla CAM (Content Addressable Memory) de un switch y qué sucede cuando se satura.
2.  **Ejecutar un ataque de saturación:** Utilizar herramientas de auditoría (Kali Linux) para forzar un desbordamiento de memoria en el switch.
3.  **Implementar defensas:** Configurar **Port Security** en equipos Cisco para detectar, bloquear y mitigar ataques de capa 2 automáticamente.
4.  **Analizar el tráfico:** Observar el comportamiento de "Fail-Open" donde el switch actúa como un Hub.

---

## 🛠️ Requisitos del Entorno (¡Importante!)

Para replicar este laboratorio fielmente, **se recomienda encarecidamente utilizar GNS3** en lugar de Cisco Packet Tracer.

### ¿Por qué GNS3 y no Packet Tracer?
* **Packet Tracer:** Es un simulador lógico. No permite la inyección de tráfico real desde máquinas externas ni soporta herramientas de pentesting reales.
* **GNS3:** Son emuladores que permiten virtualizar el sistema operativo real de Cisco (IOS) y conectarlo directamente a máquinas virtuales reales (como Kali Linux) mediante puentes de red. Esto es **obligatorio** para que el comando `macof` funcione contra el switch.

**Infraestructura Sugerida:**
* **Atacante:** Máquina Virtual con Kali Linux (Herramienta `dsniff`).
* **Infraestructura:** Switch Cisco L2 (Imagen IOU o IOSv en GNS3).
* **Víctima:** PC Virtual (VPCS o Windows) para generar tráfico legítimo.

---

## 🧠 Fundamento Teórico: CAM Table Overflow

Los switches utilizan una tabla de memoria (CAM) para mapear direcciones MAC a puertos físicos. Esta memoria es finita.

El ataque de **MAC Flooding** consiste en inundar el switch con miles de tramas falsas (con MACs de origen aleatorias). Cuando la tabla CAM se llena (Desbordamiento):
1.  El switch ya no puede aprender nuevas direcciones legítimas.
2.  Entra en modo **Fail-Open**.
3.  Comienza a realizar **Unknown Unicast Flooding**: retransmite todo el tráfico por todos los puertos (como un Hub antiguo), permitiendo al atacante interceptar datos privados (Sniffing).

---

## 🚀 Guía Paso a Paso

### ⚔️ Fase 1: El Ataque (Kali Linux)

Usaremos la suite `dsniff` para generar la inundación.

1.  **Instalación:**
    ```bash
    sudo apt-get update
    sudo apt-get install dsniff
    ```

2.  **Ejecución:**
    Lanzamos 20,000 paquetes aleatorios a través de la interfaz `eth0` hacia el switch.
    ```bash
    sudo macof -i eth0 -n 20000
    ```
    *En este punto, si no hay defensa, la consola del switch se volverá lenta y el tráfico de otros usuarios será visible en Wireshark.*

### 🛡️ Fase 2: La Defensa (Cisco IOS)

Configuramos **Port Security** para limitar el número de dispositivos permitidos por puerto.

**Comandos en el Switch (Interfaz conectada al atacante):**

```bash
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
! Habilita la seguridad del puerto
Switch(config-if)# switchport port-security
! Establece el máximo de direcciones MAC permitidas (Ej: 1 sola PC)
Switch(config-if)# switchport port-security maximum 1
! Memoriza la MAC actual (Sticky) para no tener que escribirla manual
Switch(config-if)# switchport port-security mac-address sticky
! Acción a tomar si se detecta una violación (Apagar el puerto)
Switch(config-if)# switchport port-security violation shutdown
```
---
## Validación y Resultados
Comportamiento Esperado
Al lanzar el ataque macof (o conectar un Hub con múltiples PCs) contra el puerto protegido:
El Switch detectará que ingresan más de 1 dirección MAC por el puerto Fa0/1.
Bloqueará inmediatamente la interfaz, cambiando su estado a Error-Disabled (LED apagado/rojo).
Se cortará la conexión del atacante, protegiendo la red.
