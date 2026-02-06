# Ataque 3: MD5 Hash Cracking 
## 1. Introducción:
Este laboratorio demuestra cómo el uso de contraseñas débiles y algoritmos obsoletos puede comprometer la seguridad de un sistema, incluso sin realizar ataques complejos
  solo mediante el uso de herramientas sencillas como CrackStation y el uso de un algoritmo sencillo pero muy usado todavía en sistemas antiguas o inseguros como lo es MD5.
## 2. Conceptos clave:
- **Hash:** Función matemática que toma una entrada y devuelve una cadena de caracteres alfanuméricos de longitud fija
- **MD5:** Algortimo de hashing que genera un valor de 128 bits, inseguro y obsoleto.
- **CrackStation:** Herramienta web con base de datos de hashes conocidos

## 3. Procedimiento:
1. Crear una contraseña sencilla: 12345
2. Generar su hash MD5 mediante el comando md5sum en Linux
<img width="576" height="47" alt="image" src="https://github.com/user-attachments/assets/37b62f7b-aea8-41ab-a44e-36c4aac7b929" />

3. Acceder a Crackstation mediante el link: https://crackstation.net/, pegar el hash y obtener el resultado
<img width="1281" height="508" alt="image" src="https://github.com/user-attachments/assets/5c9eec26-cd30-4039-bf29-3758e6608fd6" />

## 4. Análisis:
Es común que este tipo de hashes tengan herramientas que ayuden a obtener el hash fácilmente (cracking), esto se realiza mediante una técnica conocida como Rainbow Tables. 
Es cuando se tiene una base de datos con una cantidad enorme de hashes y su equivalente crackeado, por lo cual se va comparando con uno similar y se rompe el hash.  
<br>
Para evitar este tipo de inconvenientes se recomiendas usar algoritmos de hashing más complejos como SHA256 (estándar actual) o bcrypt, que son algoritmos con una longitud 
más larga por lo cual existen más combinaciones posibles, y es exponencialmente más difícil crackear un hash de tal magnitud (hasta ahora no se ha logrado).

## 5. Conclusión
Este ataque demuestra que al asegurar los datos existen dos variables que se deben tener en cuenta para evitar el robo masivo de datos: La primera es que se debe usar un algoritmo moderno por el cual sea computacionalmente a día de hoy lograr crackear el hash. La segunda variable es la complejidad de la contraseña, pues si el algoritmo es complejo pero no la contraseña igualmente se puede obtener mediante ataques como Rainbow Table.
