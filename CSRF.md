## __Summary__

- [__What is CSRF__](#What-is-CSRF)
- [__Manual exploitation of CSRF__](#Manual-exploitation-of-CSRF)
- [__Automatic exploitation__](#Automatic-exploitation)

# __What is CSRF__

Cross Site Request Forgery (CSRF) ocurre cuando un usuario visita una página en un sitio, que realiza una acción en un sitio diferente. Por ejemplo, digamos que un usuario hace clic en un enlace a un sitio web creado por un atacante, en el sitio web habría una etiqueta html como `<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net ">` que cambiaría el correo electrónico de la cuenta en el sitio web vulnerable a `pwned@evil-user.net`. CSRF funciona porque es la víctima quien realiza la solicitud, no el sitio, por lo que todo lo que ve el sitio es un usuario normal que realiza una solicitud normal.

Esto abre la puerta a que la cuenta del usuario se vea comprometida por completo mediante el uso de un password reset, por ejemplo. No se puede exagerar la gravedad de esto, ya que permite que un atacante obtenga potencialmente información personal sobre un usuario, como los detalles de la tarjeta de crédito (caso extremo).

# __Manual exploitation of CSRF__

![P6oMiiZ](https://user-images.githubusercontent.com/88755387/135119002-337c8115-e95c-4eed-a321-9501fa71011b.png)

Parece bastante simple. Como usuario Bob, puedo enviar fondos a Bob o Alice con cualquier saldo disponible en mi cuenta. Echemos un vistazo más de cerca a la solicitud en Burp.

![RYEnGqy](https://user-images.githubusercontent.com/88755387/135119351-441c3f70-3da4-4c89-8bda-ea3c0b04dfe5.png)

Esto se ve bien, existen parámetros que podemos personalizar y una cookie de sesión que se configura automáticamente. Todo parece vulnerable a CSRF. Intentemos crear un sitio vulnerable.

Poniendo `<img src="http://localhost:3000/transfer?to=alice&amount=100">` en un archivo HTML y usar SimpleHTTPServer o http.server para alojarlo debería cambiar el saldo de Alice en 100.

![wUUlax4](https://user-images.githubusercontent.com/88755387/135120002-f90951a9-6d72-4a2b-a5f1-636564ef9a41.png)

Podemos comprobar como, efectivamente, se han añadido 100 al sueldo de Alice.

# __Automatic exploitation__

Si queremos automatizar nuestra búsqueda existe un buen escáner, que prueba si un sitio es vulnerable a CSRF. Esta herramienta se conoce como `xsrfprobe` y se puede instalar a través de pip usando `pip3 install xsrfprobe`.

La sintaxis del comando es `xsrfprobe -u <url>/<endpoint>`. Vamos a lanzarlo contra nuestro sitio vulnerable.

![5gx7k8D](https://user-images.githubusercontent.com/88755387/135122095-aa0f367c-ef70-4cea-a1af-59b4881bbc9a.png)

El resultado confirma que hemos logrado explotarlo manualmente y que el sitio es vulnerable a CSRF.

https://github.com/0xInfection/XSRFProbe

Aqui os dejo la herramienta por si quereis echarle un ojo.
