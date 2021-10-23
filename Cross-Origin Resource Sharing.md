![image](https://user-images.githubusercontent.com/88755387/138538449-74701bca-9864-4fb2-8c23-ab07a4e300b7.png)


## __Summary__

- [__Main info__](#Control-de-acceso-HTTP-(CORS))
- [__Funcionamiento__](#Funcionamiento)
- [__Cabeceras clave de CORS__](#Cabeceras-clave-de-CORS)
- [__CORS Misconfigurations__](#CORS-Misconfigurations)
    - [__Explotación de wildcards (*) mal configurados en las cabeceras CORS__](#Explotación-de-wildcards-(*)-mal-configurados-en-las-cabeceras-CORS)
    - [__Confiar en el wildcard pre-domain como origen__](#Confiar-en-el-wildcard-pre-domain-como-origen)
- [__Mitigaciones__](#Mitigaciones)

# __Control de acceso HTTP (CORS)__

El Intercambio de Recursos de Origen Cruzado (CORS) es un mecanismo que utiliza cabeceras HTTP adicionales para permitir que un User-Agent obtenga permiso para acceder a recursos seleccionados desde un servidor, en un origen distinto (dominio) al que pertenece. Un agente crea una petición HTTP de origen cruzado cuando solicita un recurso desde un dominio distinto, un protocolo o un puerto diferente al del documento que lo generó.

Por defecto, los navegadores permiten enlazar hacia documentos situados en todo tipo de dominios si lo hacemos desde el HTML o desde Javascript utilizando la API DOM (que a su vez está construyendo un HTML). Sin embargo, no ocurre lo mismo cuando se trata de `peticiones HTTP asíncronas` mediante Javascript (AJAX), sea a través de XMLHttpRequest, de fetch o de librerías similares para el mismo propósito.

Utilizando este tipo de peticiones asíncronas, los recursos situados en dominios diferentes al de la página actual no están permitidos por defecto. Es lo que se suele denominar protección de CORS. Su finalidad es dificultar la posibilidad de añadir recursos ajenos en un sitio determinado.

Os dejo una imagen sacada de la siguiente [página](https://lenguajejs.com/javascript/peticiones-http/cors/) el cual muestra, de forma muy sencilla, el funcionamiento de este tipo de protección.

![image](https://user-images.githubusercontent.com/88755387/138522571-842178d1-c727-4691-b375-3275a46b979b.png)

# __Funcionamiento__

En el caso más sencillo, el navegador script realiza una solicitud GET para obtener un recurso de un servidor que se encuentra en otro dominio. En función de cómo sea la configuración de CORS de dicho servidor, si la solicitud proviene de un dominio con permiso para enviar solicitudes GET, el servidor de orígenes cruzados responderá devolviendo el recurso que se ha solicitado.

Si el dominio de solicitud o el tipo de solicitud HTTP no está autorizado, se denegará la solicitud. Sin embargo, con CORS es posible realizar una solicitud preliminar antes de enviarla realmente. En dicho caso, se realiza una solicitud preliminar en la que se envía la operación de solicitud de acceso OPTIONS. Si la configuración CORS del servidor de origen otorga acceso al dominio que realiza la solicitud, el servidor enviará una respuesta preliminar que contenga una lista de todos los tipos de solicitud HTTP que el dominio que realiza la solicitud puede hacer al recurso solicitado.

![image](https://user-images.githubusercontent.com/88755387/138523349-c61fa087-99f7-4723-ade5-747773b6bf2b.png)


# __Cabeceras clave de CORS__

Hay una serie de cabeceras HTTP relacionadas con CORS, pero las siguientes tres cabeceras de respuesta son las más importantes para la seguridad:

- `Access-Control-Allow-Origin` especifica qué dominios pueden acceder a los recursos de un dominio. Por ejemplo, si requester.com quiere acceder a los recursos de provider.com, los desarrolladores pueden utilizar esta cabecera para conceder de forma segura a requester.com el acceso a los recursos de provider.com.

- `Access-Control-Allow-Credentials` especifica si el navegador enviará o no cookies con la solicitud. Las cookies sólo se enviarán si la cabecera allow-credentials está configurada como true.

- `Access-Control-Allow-Methods` especifica qué métodos de solicitud HTTP (GET, PUT, DELETE, etc.) pueden utilizarse para acceder a los recursos. Esta cabecera permite a los desarrolladores mejorar la seguridad especificando qué métodos son válidos cuando requester.com solicita acceso a los recursos de provider.com.

# __CORS Misconfigurations__

## __Explotación de wildcards (*) mal configurados en las cabeceras CORS__

Cuando se tratan de malas configuraciones de CORS, uno de los ejemplos más comunes es el uso incorrecto de wildcards como `*` bajo los cuales se permite a los dominios solicitar recursos. Esto suele estar configurado por defecto, lo que significa que cualquier dominio puede acceder a los recursos de este sitio. Por ejemplo, veamos la siguiente solicitud:

```
GET /api/userinfo.php
Host: www.victim.com
Origin: www.victim.com
```

Cuando se envía la solicitud anterior, se obtiene una respuesta con la configuración de la cabecera Access-Control-Allow-Origin. Vamos a ver el código de respuesta.

```
HTTP/1.0 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

En este caso, la cabecera está configurada con un wildcard (*). Esto significa que cualquier dominio puede acceder a los recursos. 

En la siguiente imagen, modificamos el origen de la petición del dominio de la víctima al dominio del atacante (en pocas palabras, enviar datos sensibles a un servidor externo).

![image](https://user-images.githubusercontent.com/88755387/138537852-c795c352-1fdc-411a-9b7d-bffc69fc0be4.png)

A continuación se muestra la respuesta que recibimos. El wildcard se muestra en la respuesta de la cabecera de origen con `Access-Control-Allow-Origin: *`, lo que significa que el dominio víctima permite el acceso a los recursos de todos los sitios.

Como el sitio comparte información de cualquier sitio, podemos ir más alla y explotarlo utilizando nuestro propio dominio. Creamos nuestro dominio llamado https://testing.aaa.com, y lo incrustamos con código de explotación para robar la información confidencial de la aplicación vulnerable. Cuando las víctimas abran https://testing.aaa.com en el navegador, éste recupera la información confidencial y la enviará al servidor del atacante.

## __Confiar en el wildcard pre-domain como origen__

Otra forma común de explotar la mala configuración de CORS es permitiendo compartir información con nombres de dominio que están parcialmente validados. Por ejemplo, veamos la siguiente request:

```
GET /api/userinfo.php
Host: provider.com
Origin: requester.com
```

La respuesta a la anterior request sería:

```
HTTP/1.0 200 OK
Access-Control-Allow-Origin: requester.com
Access-Control-Allow-Credentials: true
```

Consideramos que el desarrollador ha configurado CORS para validar la URL de la "Origin header", con el dominio de la lista blanca como sólo "requester.com". Ahora, cuando el atacante cree la solicitud como la siguiente:

```
GET /api/userinfo.php
Host: example.com
Connection: close
Origin: attackerrequester.com
```

 El servidor respondería con:

 ```
HTTP/1.0 200 OK
Access-Control-Allow-Origin: attackerrequester.com
Access-Control-Allow-Credentials: true
 ```

 La razón por la que esto sucede es una posible validación mal configurada del backend como esta:

 ```php
 if ($_SERVER[‘HTTP_HOST’] == ‘*requester.com’)
 {
//Access data
else{ //unauthorized access }
}
 ```

 Esto puede ser explotado de la misma manera que lo hicimos para la primera misconfiguration de CORS. Podemos crear un nuevo dominio con el nombre que consiste en el nombre de dominio de la lista blanca. Luego, incrustar ese sitio malicioso con exploits que obtendrán información sensible del sitio de la víctima. 

# __Mitigaciones__

Para implementar CORS de forma segura, necesitas asociar una lista de validación `whitelist` con `Access-Control-Allow-Origin` que identifique qué dominios específicos (por ejemplo, los dominios de nuestra empresa) pueden acceder a los recursos. Entonces dicha aplicación puede validar contra esta lista cuando un dominio solicita acceso. Tampoco querremos definir su cabecera Access-Control-Allow-Origin como NULL, ya que un atacante puede enviar una solicitud con un origen NULL que eluda otros controles.

Del mismo modo, con `Access-Control-Allow-Methods` debemos especificar exactamente qué métodos son válidos para que los dominios aprobados los utilicen. Algunos pueden necesitar sólo ver los recursos, mientras que otros necesitan leerlos y actualizarlos, etc.

Es bastante fácil para un atacante configurar un visor de tráfico y observar qué solicitudes están pasando de ida y vuelta desde su sitio y cuáles son las respuestas. A partir de esto, pueden determinar si nuestro sitio es vulnerable a un ataque basado en CORS.

Por lo tanto, debemos validar todos y cada uno de los dominios que solicitan los recursos de nuestro sitio, así como los métodos que otros dominios pueden utilizar si se conceden nuestras solicitudes de acceso. Podemos identificar fácilmente las vulnerabilidades de seguridad de CORS revisando las cabeceras anteriores en la respuesta de la aplicación y validando los valores de dichas cabeceras. El uso de escáneres de código abierto es también una gran manera de descubrir las vulnerabilidades de seguridad CORS.




