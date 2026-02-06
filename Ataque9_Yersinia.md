# Ataque 9: Yersinia - DHCP Starvation

## 1. Introduccion 
Este ataque debe realizarse únicamente en entornos de laboratorio, con redes aisladas y autorización expresa.
En una red local, existen dos metodos para asignar IPs a los usuarios que se conectan, una es de manera manual donde el usuario asigna su propia IP lo cual termina siendo 
peligroso pues puede que otro usuario tome la misma IP del pool y de esta manera puede haber incoherencias como que el trafico no se envie a ninguno de los usuarios o se 
le envie a ambos. La segunda forma es mediante DHCP, este es un servidor que asigna automaticamente la IP a cada dispositivo que se conecte a la red, este mismo servidor 
gestiona las IPs para evitar duplicados y que cada dispositivo tenga una IP propia, siendo de esta forma el metodo mas usado actualmente en redes locales.
## 2. Conceptos clave
- **Pool:** IPs disponibles que tiene la red para asignar a cada nuevo dispositivo  
- **Starvation:** Agotamiento de recursos  
- **DHCP Discover:** Solicitud inicial del cliente  
- **MAC Spoofing:** Simulación de múltiples clientes  
- **Yersinia:** Herramienta para pruebas de protocolos de red
## 3. Topología del laboratorio
<img width="690" height="538" alt="image" src="https://github.com/user-attachments/assets/569525b5-f5fa-4b7c-9280-8dba16dbfb0d" />

## 4. Entorno de laboratorio
- Red local de pruebas
- Máquina víctima Ubuntu (Red Interna y Host-Only)
- Máquina atacante Kali Linux con Yersinia (Red Interna y Host-Only)

## 5. Instalación Entorno Gráfico Yersinia (Opcional):
Para la instalación de Yersinia se requiere instalar dependencias gráficas para ello se corre los comandos:
1. Actualizar apt
``` bash
sudo apt update
```
2. Instalar dependencias gráficas (gtk-3)
``` bash
apt install libgtk-3-dev libglib2.0-dev libpcap-dev
```
3. Clonar el repositorio de la herramienta
``` bash
git clone https://github.com/tomac/yersinia.git
```
4. Dirigirse a la carpeta creada
``` bash
cd yersinia
```
5.  Ir a la configuración
``` bash
./configure
```
6.  Compilar
``` bash
make
```
``` bash
make install
```
## 6. Procedimiento
### Fase 1: Ingresar a la herramienta
1. Iniciar la interfaz de Yersinia (con permisos sudo)
``` bash
yersinia -I
```
2. En la interfaz para pasar a DHCP se presiona f2
3. Luego, se presiona x para ver los ataques
### Fase 2: Ejecutar ataque y analizar el proceso
4. Antes de ejecutar el ataque, se abre Wireshark para observar como funciona el envío de paquetes DHCP, la interfaz es eth0 (por defecto)  
4.1. Dentro de Wireshark, filtro por "dhcp" para evitar cualquier ruido que no sea DHCP
<img width="1919" height="892" alt="image" src="https://github.com/user-attachments/assets/9ec98bb6-7818-4e71-84a3-62b5a275f936" />

5. Se escoge el sending DISCOVER packet (modelo DORA de DHCP) que es la opción 1 (presionar 1)
6. En la herramienta de Yersinia se puede visualizar la cantidad de paquetes que se envían

7. Y en Wireshark igualmente se puede visualizar la cantidad colosal de paquetes que se están enviando

### Fase 3: Crear nuevo servidor DHCP (redirección de paquetes a atacante)
8. Una vez realizado el ataque con la herramienta, se para presionando X, luego se abre el menú de ataques otra vez en DHCP y esta vez se selecciona la opción 2 que es para que el tráfico nuevo DHCP se redirija a la máquina actual
9. En este menú se selecciona lo siguiente:
- Server ID: IP de la máquina atacante (la que se está usando)
- Start IP: Donde comienza el rango de IPs en el pool
- End IP: Última IP del pool
- Lease Time (secs): Cada cuanto tiempo se puede solicitar una nueva IP
- Renew: Obtener una nueva IP
- Subnet Mask
- Router: Puerta de entrada para víctimas (IP del atacante)
- DNS Server: Resolución DNS (IP de un DNS de confianza)
- Domain: Nombre cualquiera para identificar servidor
10. Se crea el servidor, la imagen a continuación es un ejemplo de datos del servidor:

## 7. Conclusión
Los servidores DHCP en una red local pueden representar un riesgo si no se controlan adecuadamente, pues un atacante puede realizar un ataque DDoS y cambiar el flujo del tráfico de red hacia su dispositivo, lo cual deja vulnerable la información de los usuarios conectados a la misma red.

