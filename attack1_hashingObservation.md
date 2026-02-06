# Ataque 1: Observación de Hashing en Linux

## Objetivo

Comprender cómo se almacenan las contraseñas en sistemas Linux mediante funciones hash y el uso de salt para aumentar la seguridad, observando su estructura y componentes.

---

## Herramientas Utilizadas

- **Sistema Operativo:** Linux
- **Herramientas:** Terminal, OpenSSL
- **Archivos del sistema:** `/etc/shadow`

---

## Conceptos Previos

### ¿Qué es un Hash?

En Linux, las contraseñas **no se almacenan en texto plano**, sino como hashes dentro del archivo `/etc/shadow`.

- Un **hash** es el resultado de aplicar una función criptográfica unidireccional a una contraseña.
- Es prácticamente imposible revertir un hash para obtener la contraseña original.

### ¿Qué es un Salt?

El **salt** es un valor aleatorio que se agrega a la contraseña antes de aplicar la función hash. Esto previene:

- **Ataques con tablas rainbow** (tablas precalculadas de hashes)
- **Identificación de contraseñas idénticas** entre usuarios

---

## Parte 1: Creación de Usuario y Contraseña

### Paso 1: Crear un nuevo usuario

Se crea un usuario para el laboratorio de pruebas con el siguiente comando:
```bash
sudo adduser usuario_lab
```

### Paso 2: Asignar una contraseña

Durante la creación del usuario, se asigna la siguiente contraseña:
```
Password123
```
---

## Parte 2: Identificación del Hash ID y el Salt

### Paso 1: Acceder al archivo de contraseñas

El archivo donde se almacenan los hashes de las contraseñas es `/etc/shadow`. Para visualizarlo se usa el comando:
```bash
sudo cat /etc/shadow
```

### Paso 2: Buscar el usuario creado

Localizar la línea correspondiente a `usuario_lab`:
```bash
sudo grep usuario_lab /etc/shadow
```

### Paso 3: Identificar los componentes del hash

Una entrada típica del archivo `/etc/shadow` tiene la siguiente estructura:
```
usuario:$5$salt$hash:18500:0:99999:7:::
```

#### Estructura detallada del campo de contraseña:

| Parte | Significado |
|-------|-------------|
| `$5$` | Identificador del algoritmo (SHA-256 Crypt) |
| `salt` | Valor aleatorio que protege el hash |
| `hash` | Resultado de la función hash |

#### Ejemplo real:
```
usuario_lab:$5$Xk93Lw$Qe7T0b2FJk1m8N9pQ3rS5tU7vW9xY1zA3bC5dE7fG9h:18500:0:99999:7:::
```

---

## Parte 3: Creación de una Contraseña con Salt Personalizado

### Paso 1: Generar un hash manualmente con OpenSSL

Se utiliza el comando `openssl` para generar un hash con un salt definido manualmente:
```bash
openssl passwd -5 -salt Xk93Lw Password123
```

**Parámetros:**
- `-5`: Usa el algoritmo SHA-256 Crypt
- `-salt Xk93Lw`: Define el salt manualmente
- `Password123`: La contraseña a hashear

### Paso 2: Resultado esperado

El comando generará una salida similar a:
```
$5$Xk93Lw$Qe7T0b2FJk1m8N9pQ3rS5tU7vW9xY1zA3bC5dE7fG9h
```

### Paso 3: Comparación con /etc/shadow

Para verificar que el hash generado coincide con el almacenado:
```bash
# Ver el hash en /etc/shadow
sudo grep usuario_lab /etc/shadow | cut -d: -f2

# Comparar con el hash generado manualmente
openssl passwd -5 -salt Xk93Lw Password123
```

### Paso 4: Verificación

Se comprueba que:

- ✅ La **misma contraseña**
- ✅ Con el **mismo salt**
- ✅ Produce **exactamente el mismo hash**

Esto demuestra que el proceso de hashing es determinístico cuando se usan los mismos parámetros.

---

## Anatomía de una Contraseña Hasheada

### Ejemplo de estructura:
```
$5$Xk93Lw$Qe7T0b2FJk1m8N9pQ3rS5tU7vW9xY1zA3bC5dE7fG9h
```

| Componente | Valor | Significado |
|------------|-------|-------------|
| **Algoritmo** | `$5$` | SHA-256 Crypt |
| **Salt** | `Xk93Lw` | Valor aleatorio único |
| **Hash** | `Qe7T0b2FJk1...` | Resultado final del hash |

---

## Algoritmos Hash Según su Identificador

| Identificador | Algoritmo | Nivel de Seguridad |
|---------------|-----------|-------------------|
| `$1$` | MD5 Crypt | ❌ Obsoleto (inseguro) |
| `$5$` | SHA-256 Crypt | ✅ Seguro |
| `$6$` | SHA-512 Crypt | ✅ Más seguro (recomendado) |
| `$y$` | yescrypt | ✅ Moderno (más reciente) |

> **Recomendación:** En sistemas modernos, se prefiere SHA-512 (`$6$`) o yescrypt por su mayor seguridad.

---

## Experimentación Adicional

### Generar hashes con diferentes salts
```bash
# Hash 1 con salt aleatorio
openssl passwd -5 Password123

# Hash 2 con salt aleatorio
openssl passwd -5 Password123

# Observar que son diferentes aunque la contraseña sea la misma
```

### Generar hash con SHA-512
```bash
openssl passwd -6 -salt TestSalt Password123
```

### Modificar la contraseña de un usuario manualmente
```bash
# 1. Generar el hash
NEW_HASH=$(openssl passwd -6 -salt RandomSalt NuevaPassword456)

# 2. Editar /etc/shadow (con mucho cuidado)
sudo vipw -s
```

---

## Preguntas de Reflexión

1. **¿Por qué dos usuarios con la misma contraseña tienen hashes diferentes?**
   - Porque cada hash usa un salt diferente generado aleatoriamente.

2. **¿Qué pasaría si no existiera el salt?**
   - Las contraseñas idénticas tendrían el mismo hash, facilitando ataques con tablas rainbow.

3. **¿Se puede obtener la contraseña original a partir del hash?**
   - No directamente. Solo mediante fuerza bruta o diccionarios se puede intentar encontrar una coincidencia.

---

## Medidas de Seguridad Observadas

1. ✅ **Hashing unidireccional**: No es posible revertir el hash a la contraseña original
2. ✅ **Uso de salt**: Previene ataques con tablas precalculadas
3. ✅ **Algoritmos robustos**: SHA-256/SHA-512 son resistentes a colisiones
4. ✅ **Archivo protegido**: `/etc/shadow` solo es accesible por root

---

## Conclusiones

Este laboratorio ha demostrado cómo Linux protege las contraseñas mediante:

- **Funciones hash criptográficas** que transforman contraseñas en valores irreversibles
- **Salt único** para cada contraseña que previene ataques precomputados
- **Almacenamiento seguro** en `/etc/shadow` con permisos restrictivos

### Implicaciones de Seguridad

- **Para administradores**: Verificar que el sistema use algoritmos seguros (`$6$` o `$y$`)
- **Para usuarios**: Comprender que contraseñas débiles pueden ser crackeadas con fuerza bruta
- **Para atacantes potenciales**: El salt dificulta significativamente los ataques con tablas rainbow
