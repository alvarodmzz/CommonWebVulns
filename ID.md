![debug](https://user-images.githubusercontent.com/88755387/130437670-0f29ad5a-79d4-41f1-9668-fb0f4c426d53.jpg)

## __<span style="color:white"> Tabla de contenidos </span>__

- [__<span style="color:gold"> Información principal </span>__](#What-is-information-disclosure?)
- [__<span style="color:aqua"> Tipos de ID </span>__](#Different-Types-of-Information-Disclosure-Issues)
  - [<span style="color:aqua"> Banner grabbing </span>](#Banner-grabbing)
  - [<span style="color:aqua"> Source Code Disclosure </span>](#Source-Code-Disclosure)
  - [<span style="color:aqua"> Filename and File path disclosure </span>](#Filename-and-File-path-disclosure)
  - [<span style="color:aqua"> Inappropriate handling of sensitive data </span>](#Inappropriate-handling-of-sensitive-data)
- [__<span style="color:violet"> NTLM </span>__](#NTLM)
- [__<span style="color:springgreen"> Protección frente ID's </span>__](How-to-prevent-Information-Disclosure-attacks)

# __<span style="color:gold"> Qué es "Information Disclosure"? </span>__

> La divulgación de información se produce cuando una aplicación web no protege adecuadamente la información confidencial, lo que provoca que se revele información o datos sensibles de los usuarios o cualquier cosa relacionada con ellos a un tercero.

Este tipo de problemas no se pueden explotar en la mayoría de los casos, pero se consideran problemas de seguridad de aplicaciones web porque permiten a los atacantes recopilar información relevante que se puede utilizar más adelante para lograr un mayor impacto en el ataque.

# __<span style="color:aqua"> Diferentes tipos de Information Disclosure </span>__

## __<span style="color:aqua"> Banner grabbing </span>__

> El banner grabbing es un proceso de recopilación de información como el sistema operativo, los detalles del servidor, el nombre del servicio que se ejecuta con su número de versión y mucha información al respecto.

En la mayoría de los casos, no implica la filtración de información crítica, sino más bien información que puede ayudar al atacante a través de la fase de explotación del ataque. Por ejemplo, si el objetivo filtra la versión de PHP que se está ejecutando en el servidor y resulta ser vulnerable a un Remote Code Execution (RCE) porque no se actualizó, los atacantes pueden aprovechar la vulnerabilidad conocida y tomar el control total de la aplicación web.

### __Example__

Podemos ver que Netsparker identificó una versión antigua de PHP ejecutándose en el host de destino. También puede usar la pestaña HTTP Request/Response en Netsparker para ver la respuesta HTTP del servicio de destino en formato sin procesar, en el que también se resalta la versión anterior de PHP.

![http_response_php_version](https://user-images.githubusercontent.com/88755387/130510165-a1735cc3-e546-442c-acf6-44f215570733.png)

## __<span style="color:aqua"> Source Code Disclosure </span>__

> Esto ocurre cuando el código del entorno de back-end de una aplicación web se expone al público. Si se revelan archivos de código fuente, un atacante puede usar dicha información para descubrir fallas lógicas.

### __Ejemplo__

Los navegadores web saben cómo analizar la información que reciben del encabezado HTTP Content-Type que envía el servidor web en la respuesta HTTP. Por ejemplo, en la captura de pantalla que os proporciono a continuación, podemos ver que el encabezado Content-Type está configurado en text/html, por lo que el navegador analiza el HTML y muestra la salida.

![mime_type_http_headerpng](https://user-images.githubusercontent.com/88755387/130511061-bf802e87-2700-4a59-b10c-86c5a5a4350f.png)

Aunque si el servidor web está mal configurado y, por ejemplo, envía el encabezado Content-Type: text/plain en lugar de Content-Type: text/html al servir una página HTML, el código se mostrará como texto sin formato en el navegador, lo que permite al atacante para ver el código fuente de la página.

## __<span style="color:aqua"> Filename and File path disclosure </span>__

> Esto puede suceder debido a un manejo incorrecto de la entrada del usuario, excepciones en el back-end o una configuración inapropiada del servidor web. En ocasiones, dicha información se puede encontrar o identificar en las respuestas de las aplicaciones web, páginas de error, información de depuración, etc.

### __Ejemplo__

Una prueba simple que un atacante puede hacer para verificar si la aplicación web revela algún nombre de archivo o ruta es enviar una cantidad de solicitudes diferentes que él cree que el objetivo podría manejar de manera diferente. Por ejemplo, al enviar la solicitud a continuación, la aplicación web devuelve una respuesta 403 (Prohibido):

`https://www.example.com/%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd`

Pero cuando el atacante envía la siguiente secuencia, obtiene una respuesta 404 (No encontrado):

`https://www.example.com/%5C../%5C../%5C../%5C../%5C../%5C../etc/doesntexist`

Dado que para la primera solicitud el atacante obtuvo un error 403 y para la segunda obtuvo un 404, sabe que en el primer caso el archivo en cuestión existe. Por lo tanto, el atacante puede utilizar el comportamiento de la aplicación web a su favor, como un exploit para comprender cómo está estructurado el servidor y verificar si una determinada carpeta o archivo existe en el sistema.

## __<span style="color:aqua"> Inappropriate handling of sensitive data </span>__

> Esto puede suceder cuando los datos confidenciales no se eliminan del código fuente o de otro lugar. Algunos datos como el nombre de usuario, la contraseña o algún comentario importante pueden estar presentes allí, lo que puede revelar algunos datos confidenciales.

# __<span style="color:violet"> NTLM </span>__

> De forma predeterminada, en el entorno de dominio de Windows, la autenticación utiliza el protocolo Kerberos para autenticar y autorizar a los usuarios. Si el protocolo Kerberos no se puede utilizar por algún motivo, se utiliza NT LAN Manager (NTLM) como reserva. Podemos deshabilitar este comportamiento estableciendo la propiedad `AllowNtlm` en `false`. 

Los problemas a tener en cuenta al permitir NTLM son los siguientes:

- Expone el nombre de usuario del cliente. Si es necesario mantener la confidencialidad del nombre de usuario tenemos que establecer la propiedad `AllowNTLM` del enlace en `false`.
- No proporciona autenticación de servidor. Por lo tanto, el cliente no puede asegurarse de que se está comunicando con el servicio correcto cuando se usa como protocolo de autenticación.

## __<span style="color:violet"> Especificar las credenciales del cliente o la identidad no válida obliga al uso de NTLM </span>__

Al crear un cliente, especificar sus credenciales sin un nombre de dominio o especificar una identidad de servidor no válida, hace que se utilice NTLM en lugar del protocolo Kerberos (si la propiedad `AllowNtlm` está configurada en `true`). 

Debido a que NTLM no realiza la autenticación del servidor, la información puede potencialmente divulgarse.

Por ejemplo, es posible especificar las credenciales de cliente de Windows sin un nombre de dominio, como se muestra en el siguiente código de Visual C#.

```C#
MyChannelFactory.Credentials.Windows.ClientCredential = new System.Net NetworkCredential("username", "password");
```
El código no especifica un nombre de dominio y, por lo tanto, se utilizará NTLM.

Si se especifica el dominio, pero también se especifica un nombre principal de servicio no válido mediante la función de identidad de punto final, se utilizará NTLM.

# __<span style="color:springgreen"> Como prevenir los ataques de Information Disclosure </span>__

> Los problemas de seguridad en la divulgación de información pueden parecer triviales, pero no lo son. Permiten que los hackers obtengan información detallada y confidencial sobre el objetivo que quieren atacar simplemente realizando pruebas básicas y, a veces, simplemente buscando información en páginas públicas.

Las siguientes son algunos pasos a seguir para poder asegurarse de que las aplicaciones web estén bien protegidas contra los problemas de divulgación de información más obvios:

- Asegurarse de que su servidor web no envíe encabezados de respuesta o información de fondo que revele detalles técnicos sobre el tipo, la versión o la configuración de la tecnología back-end.

- Los datos confidenciales, los archivos y cualquier otro elemento de información que no necesite estar en los servidores web nunca deben cargarse en el mismo.

- No codificar credenciales, claves API, direcciones IP o cualquier otra información confidencial en el código, incluidos nombres y apellidos, ni siquiera en forma de comentarios.

- Siempre verificar si cada una de las solicitudes para crear/editar/ver/eliminar recursos tiene controles de acceso adecuados, evitando problemas de escalada de privilegios y asegurándo que toda la información confidencial permanezca tal y como su nombre indica.

- Asegurarse siempre de que existan los controles de acceso y las autorizaciones adecuados para impedir el acceso de los atacantes a todos los servidores, servicios y aplicaciones web.

- Configurar el servidor web para no permitir `directory listing` y asegurarse de que la aplicación web siempre muestre una página web predeterminada.

- Utilizar mensajes de error genéricos tanto como sea posible. No proporcionar a los atacantes pistas sobre el comportamiento de las aplicaciones innecesariamente.

- Asegurarse de que todos los servicios que se ejecutan en los puertos abiertos del servidor no revelen información sobre sus compilaciones y versiones.

- Configurar los tipos MIME correctos en el servidor web para todos los diferentes archivos que se utilizan en las aplicaciones web.




