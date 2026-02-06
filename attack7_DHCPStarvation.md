# ☠️ Laboratorio: Ataque de Inanición DHCP (DHCP Starvation)

> **DISCLAIMER:** Este laboratorio tiene fines **estrictamente educativos** para la asignatura de Seguridad en Redes. La ejecución de ataques de agotamiento de recursos (DoS) en redes no autorizadas es ilegal y puede causar la caída total de la conectividad.

## 🎯 Objetivos del Laboratorio
1.  **Analizar la vulnerabilidad:** Comprender cómo la falta de autenticación en el proceso DORA del protocolo DHCP permite ataques de agotamiento.
2.  **Ejecutar un ataque de DoS:** Utilizar la herramienta **Yersinia** para saturar el servidor DHCP (Router) con peticiones falsas.
3.  **Evidenciar el impacto:** Observar mediante **Wireshark** la inundación de tráfico y el comportamiento de *MAC Spoofing* masivo.
4.  **Implementar mitigación:** Conocer las contramedidas en equipos de conmutación (Switches) para prevenir este escenario.

---

## 🛠️ Entorno y Requisitos

### Infraestructura
* **Atacante:** Kali Linux (VM en Hyper-V configurada en **Modo Bridge**).
* **Víctima/Servidor:** Router Doméstico (Gateway).
* **Herramientas:** * `Yersinia` (Framework de ataques de Capa 2).
    * `Wireshark` (Análisis de tráfico).

### ⚙️ Configuración Previa (Crítica)
Para asegurar la estabilidad del atacante durante la denegación de servicio:
1.  Se configuró una **IP Estática** en Kali Linux.
2.  Se validó la conectividad directa a la Capa 2 (Bridge) para alcanzar el servidor DHCP real.

---

## 🧠 Marco Teórico: DHCP Starvation

El ataque aprovecha que el servidor DHCP asigna direcciones IP basándose únicamente en la dirección MAC del solicitante, sin validar su identidad.

**Mecánica del Ataque:**
1.  **Spoofing:** El atacante genera miles de direcciones MAC aleatorias por segundo.
2.  **Flooding:** Envía un paquete `DHCP DISCOVER` por cada MAC falsa.
3.  **Exhaustion:** El servidor responde con `DHCP OFFER` y reserva una IP para cada "cliente fantasma".
4.  **Denial of Service:** El pool de direcciones IP disponibles (Scope) se agota. Los usuarios legítimos nuevos no pueden obtener una IP y pierden conectividad.

---

## 🚀 Guía de Ejecución (Paso a Paso)

### 1. Preparación del Sniffer
Iniciamos Wireshark filtrando por el protocolo `bootp` para capturar solo el tráfico DHCP.

### 2. Ejecución de Yersinia (Modo Interactivo)
Debido a limitaciones de librerías gráficas, se ejecutó el ataque en modo ncurses:

```bash
sudo yersinia -I
```
---
Paso A: Tecla g -> Seleccionar modo DHCP.

Paso B: Tecla x -> Panel de Ataques.

Paso C: Seleccionar ataque sending DISCOVER packet (DoS).

Paso D: Tecla 1 -> Lanzar ataque.
### Medidas de Mitigación

Para proteger la red en un entorno empresarial (Cisco Switch), se aplican dos capas de seguridad:

1. Port Security (Limitación de MACs)
Evita que un solo puerto simule ser múltiples clientes.
