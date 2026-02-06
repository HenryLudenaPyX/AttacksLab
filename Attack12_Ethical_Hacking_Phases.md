# Ataque 12: Fases de Ethical Hacking según OWASP

## Enfoque del documento:
- ✅ Como se organiza una prueba de seguridad controlada mediante un proceso estandarizado
- ❌ Cómo "hackear" para alcanzar un objetivo dañino
## 1. Introducción:
El Ethical Hacking es un proceso estructurado cuyo objetivo es identificar vulnerabilidades de forma controlada y autorizada, siguiendo una serie de fases bien definidas. 
Existen diferentes modelos que definen procesos de pentesting, en este documento se va a presentar el modelo que recomienda OWASP.

## 2. Prerrequisitos éticos y técnicos:
En toda práctica de pentesting, se debe tener en cuenta siempre los siguientes requisitos mínimos para evitar problemas legales o éticos:
- Autorización explícita (Si es fuera del alcance del pentester): Establecer consenso y límite de que sistemas se pretende evaluar
- Entorno de laboratorio: Evitar problemas persistentes e irreversibles
- Sistemas aislados: El desarrollo no sale del marco establecido previamente
- Conocimiento básico de redes y sistemas: Reconocer cuál es la lógica de las herramientas, evitando su uso mediante el azar

## 3. Entorno de trabajo
Al trabajar en ethical hacking existen los siguientes roles que definen el alcance y la consecuencia que tiene cada uno.

### 3.1 Componentes del entorno

| Máquina | Rol | Descripción |
|-------|----|-------------|
| Máquina atacante | Auditor de seguridad | Sistema desde el cual se realizan las pruebas de análisis y evaluación |
| Máquina objetivo | Sistema bajo prueba | Equipo que simula un servidor o cliente vulnerable |
| Red virtual | Medio de comunicación | Red local aislada que permite la interacción entre las máquinas |

### 3.2 Configuración de red

- Las máquinas se encuentran conectadas a una red interna o host-only
- No existe conexión directa a Internet
- El tráfico queda limitado al entorno de laboratorio
- Se permite la captura y análisis de paquetes de red

## 4. Fases del Ethical Hacking
El Ethical Hacking se estructura en una serie de fases secuenciales cuyo objetivo es evaluar la seguridad de un sistema de forma controlada, ética y documentada. Cada fase cumple un propósito específico y prepara el terreno para la siguiente.

### Fase 1: Reconocimiento

**Objetivo:**  
Recopilar información inicial sobre el sistema objetivo sin interactuar directamente con él de forma intrusiva.

**Configuración de las máquinas:**  
- Máquina atacante configurada con herramientas de análisis pasivo  
- Sistema objetivo operativo y accesible en la red  
- Entorno aislado de laboratorio

**Pasos generales:**  
- Identificación del sistema objetivo  
- Observación del tráfico de red  
- Recolección de información pública o visible

**Resultado esperado:**  
- Información básica del objetivo  
- Identificación preliminar de la superficie de ataque

> *Nota:* Existen técnicas de análisis que pueden parecer activas pero no lo son, como reconocimiento de WAF (Web App Firewall).

---

### Fase 2: Escaneo y Enumeración

**Objetivo:**  
Identificar servicios activos, puertos abiertos y posibles vectores de ataque mediante interacción controlada con el sistema.

**Configuración de las máquinas:**  
- Máquina atacante con herramientas de escaneo habilitadas  
- Sistema objetivo accesible dentro del entorno de red  
- Conectividad estable entre las máquinas

**Pasos generales:**  
- Detección de puertos abiertos  
- Identificación de servicios y versiones  
- Enumeración de recursos disponibles

**Resultado esperado:**  
- Mapa de servicios activos  
- Lista de posibles vulnerabilidades técnicas

> *Nota:* A partir de aquí cualquier acción sin voluntad de la víctima está fuera de lo legal.

---

### Fase 3: Explotación

**Objetivo:**  
Verificar si las vulnerabilidades identificadas pueden ser explotadas de forma controlada.

**Configuración de las máquinas:**  
- Sistema objetivo con vulnerabilidad conocida  
- Máquina atacante preparada para pruebas controladas  
- Registro activo de las acciones realizadas

**Pasos generales:**  
- Selección de una vulnerabilidad específica  
- Prueba controlada de explotación  
- Validación del impacto obtenido

**Resultado esperado:**  
- Confirmación de la vulnerabilidad  
- Evidencia técnica del acceso o fallo

> *Nota:* En esta fase no se busca causar daño, sino demostrar la existencia del riesgo.

---

### Fase 4: Post-Explotación

**Objetivo:**  
Evaluar el impacto real del acceso obtenido y el alcance potencial del compromiso.

**Configuración de las máquinas:**  
- Acceso temporal y controlado al sistema  
- Monitorización activa del entorno

**Pasos generales:**  
- Evaluación de privilegios obtenidos  
- Identificación de información accesible  
- Análisis del impacto potencial

**Resultado esperado:**  
- Medición del riesgo real  
- Comprensión del alcance de la vulnerabilidad

> *Nota:* No se busca destruir el sistema sino evaluar hasta que nivel puede afectar si un atacante real actuara.

---

### Fase 5: Reporte

**Objetivo:**  
Documentar los hallazgos de forma clara, estructurada y responsable.

**Configuración de las máquinas:**  
- Evidencias recopiladas durante las fases anteriores  
- Herramientas de documentación y análisis

**Pasos generales:**  
- Descripción de vulnerabilidades encontradas  
- Evaluación del impacto  
- Propuesta de medidas de mitigación

**Resultado esperado:**  
- Informe técnico comprensible  
- Recomendaciones para mejorar la seguridad del sistema
> *Nota:* El reporte es la fase más importante del Ethical Hacking, ya que transforma el análisis técnico en valor real.

## 5. Marco Ético

El propósito del Ethical Hacking no es comprometer sistemas, sino identificar vulnerabilidades con el fin de fortalecer la seguridad.

Los principios éticos fundamentales incluyen:

- **Autorización explícita:**  
  Todas las pruebas deben realizarse únicamente sobre sistemas para los cuales se ha concedido permiso formal.

- **No causar daño:**  
  Las pruebas deben evitar la pérdida de datos, interrupciones prolongadas del servicio o daños a la infraestructura.

- **Responsabilidad profesional:**  
  El auditor debe actuar de forma transparente, documentando todas las acciones realizadas.

- **Confidencialidad:**  
  La información obtenida durante las pruebas no debe ser divulgada ni reutilizada fuera del contexto autorizado.

### 5.1. Consideraciones Legales

Desde el punto de vista legal, el acceso no autorizado a sistemas informáticos constituye un delito en la mayoría de legislaciones, independientemente de la intención.

Por ello, es imprescindible:

- Contar con un **acuerdo previo** que delimite el alcance de las pruebas  
- Definir claramente qué sistemas pueden ser evaluados  
- Respetar las leyes de protección de datos y privacidad  
- Limitar las pruebas a entornos controlados o de laboratorio

La ausencia de autorización convierte cualquier actividad técnica en una **acción ilegal**, incluso si su objetivo es educativo.

## 6. Conclusión
El Ethical Hacking no consiste en atacar sistemas, sino en comprenderlos profundamente para fortalecerlos mediante un proceso ordenado y ético. Mediante el conocimiento de esta rama de la informática se pretende reforzar los sistemas
existentes para hacerlos seguros frente a amenazas externas. De esta manera evaluando desde el puntos de vista del atacante y cómo actuar frente al casos hipotético que sucedería un ataque real.

## Links:
OWASP Pentesting Methodology: https://owasp.org/www-project-web-security-testing-guide/latest/3-The_OWASP_Testing_Framework/1-Penetration_Testing_Methodologies#open-source-security-testing-methodology-manual
OWASP Top 10 2025: https://owasp.org/Top10/2025/
