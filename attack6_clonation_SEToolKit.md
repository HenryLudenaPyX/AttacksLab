## Introducción:
Este laboratorio tiene como objetivo demostrar cómo los ataques de Ingeniería Social (SE) explotan el factor humano mediante la clonación de páginas 
web con Social Engineering ToolKit(SEToolKit), con fines exclusivamente educativos y de concienciación en ciberseguridad. 
NO USAR FUERA DE CONTEXTO ACADÉMICO

## Conceptos clave:
- **Ingeniería Social:** Técnica de manipulación psicológica que busca obtener información confidencial que compromete la seguridad de las personas.
- **SEToolKit:** Framework para clonar páginas web, pruebas de ingeniería social para evaluar concienciación

## Desarrollo:
1. En la terminal de Kali Linux con permisos de administrador (sudo) abrir la herramienta mediante el comando:
```
setoolkit
```
<img width="680" height="836" alt="image" src="https://github.com/user-attachments/assets/47804c3c-d7b2-4ca7-a5f7-2274c103c933" />

2. Escoger la opción, para realizar Ataques de SE
<img width="335" height="212" alt="image" src="https://github.com/user-attachments/assets/a26b18d0-8c2f-482a-b90b-b7c1de21144c" />

3. Luego, la opción 2, para entrar a los Vectores de ataque a sitios web
<img width="355" height="274" alt="image" src="https://github.com/user-attachments/assets/d2005e0b-274f-462e-9580-9b897c60df7d" />

4. La opción 3, para robar credenciales de la víctima mediante clonación de sitio web
<img width="341" height="186" alt="image" src="https://github.com/user-attachments/assets/2c92c158-e3e4-49f7-8304-e12470f404f4" />

5. Escoger la opción 2, para clonar sitio web
<img width="285" height="129" alt="image" src="https://github.com/user-attachments/assets/17b983ed-c563-4ffc-8601-66ee9d797bac" />

6. Se deja el campo de IP vacío (Enter) para usar la IP de la máquina que es a donde van a llegar las credenciales de la víctima. Además, se coloca
la URL de la página a clonar
<img width="656" height="77" alt="image" src="https://github.com/user-attachments/assets/bc654ac2-9d72-4626-b726-c657f486c2de" />

7. Ingresar a un navegador como Mozilla Firefox y pegar la IP de la máquina (en este caso 10.0.3.15) que es donde se aloja la página. La víctima
  ingresa sus credenciales creyendo que es el sitio web original
<img width="379" height="195" alt="image" src="https://github.com/user-attachments/assets/cbb2d609-130d-4463-9f3e-be53d3a62f95" />

9. Al regresar a la terminal se puede observar como las credenciales se han enviado a la máquina
<img width="243" height="37" alt="image" src="https://github.com/user-attachments/assets/a7ae594b-47f3-4e11-bb31-6acfd66bee21" />

## Análisis:
Este tipo de escenarios se suelen usar en contextos para capacitaciones para concientizar a los usuarios sobre lo sencillo que es falsificar una 
página web y que el atacante obtenga las credenciales fácilmente aprovechándose de la confianza de las personas.  
<br>
Desde la perspectiva del atacante en cambio, suelen usar este tipo de ataques para clonar páginas web de sitios que usan las víctimas y envían correos
o mensajes indicando que deben iniciar sesión.  
<br>
Cabe recalcar que para este ejemplo se usó una página web con un login sencillo, pues actualmente sitios web modernos usan seguridad avanzada como
uso de AJAX y cifrado, por lo cual al intentar clonar este tipo de páginas son SEToolkit lo único que devuelve es basura.

## Conclusión:
La clonación de páginas demuestra que el eslabón más débil de la seguridad no es el software, sino el comportamiento humano.
