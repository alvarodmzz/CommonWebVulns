![IDORimage](https://user-images.githubusercontent.com/88755387/130352752-8ec4bf62-5b9b-422f-bfab-2a94df527608.png)

## __Indice__

- [__Main info__](#What-is-IDOR-vulnerability?)
- [__Impact__](#Potential-Impact)
- [__Mitigations__](#Mitigations)
- [__Basic example__](#Basic-IDOR)
- [__IDOR chain__](#IDOR-chain)
- [__PortSwigger Labs__](#PortSwigger-Labs)



# __Qué es la vulnerabilidad IDOR?__

Los IDORs (Insecure Direct Object References) son un tipo de vulnerabilidad de control de acceso que se produce cuando una aplicación utiliza la entrada proporcionada por el usuario para acceder a un objeto (servidor), como un archivo, directorio o clave de base de datos directamente.

Esta vulnerabilidad es la más conocida dentro del campo de Improper Access Control el cual vamos a definir para conocer un poco más de que se trata este tipo de seguridad.

El control de acceso (Access Control) es un proceso de seguridad que controla el uso de recursos específicos dentro de un criterio predefinido y es parte del modelo de seguridad AAA (Autenticación, Autorización, Contabilidad). Todos los sistemas modernos utilizan determinados modelos de control de acceso para gestionar su seguridad. Los modelos de control de acceso se pueden agrupar en tres clases principales: control de acceso obligatorio (MAC), control de acceso discrecional (DAC) y control de acceso basado en roles (RBAC).

En resumen:

`Obtener información confidencial con solo cambiar algunos valores en el parámetro`.

## __Tipos de IDOR__

- Blind IDOR: El tipo de IDOR en el que los resultados de la explotación no se pueden ver en la respuesta del servidor. Por ejemplo, modificar los datos privados de otros usuarios sin acceder a ellos.

- Generic IDOR: El tipo de IDOR en el que se pueden ver los resultados de la explotación en la respuesta del servidor. Por ejemplo acceder a datos confidenciales o archivos pertenecientes a otro usuario.

- IDOR w/ Reference to Objects: Se utiliza para acceder o modificar un objeto no autorizado. Por ejemplo, acceder a la información de la cuenta bancaria de otros usuarios enviando una solicitud de este tipo:
`http://ejemplo.com/cuentas?id={ID de referencia}`

- IDOR w/ Reference to Files: Se utiliza para acceder a un archivo no autorizado. Por ejemplo, un servidor de chat en vivo almacena las conversaciones confidenciales en archivos con nombres como números crecientes y cualquier conversación se puede recuperar simplemente enviando solicitudes como esta: `ejemplo.com/1.log, ejemplo.com/2.log, ejemplo.com/3.log` y así sucesivamente.

# __Impactos potenciales__

Esta debilidad permite a un atacante eludir las restricciones de seguridad previstas y realizar una variedad de acciones según la fuente del error y la funcionalidad de la aplicación. Un atacante podría realizar ciertas acciones obteniendo privilegios elevados, leyendo información que de otro modo estaría restringida, ejecutando comandos, evitando los mecanismos de seguridad implementados, etc.

# __Mitigaciones__ 

Generalmente este vector de ataque resulta en errores lógicos. Los desarrolladores deben administrar cuidadosamente la configuración y el manejo de los privilegios, así como también prestar atención a las zonas de seguridad dentro de la aplicación. También deben integrar mecanismos que controlen la funcionalidad de separación de privilegios y pensar en crear una arquitectura que se base en el principio de privilegios mínimos.

A continuación os dejo una imagen donde podeis ver diferentes bypasses y tips:

![IDORTechniques-MindMap](https://user-images.githubusercontent.com/88755387/130352760-78af2a2d-e86b-4f74-a6ec-71fd83eacacb.png)

Ahora que ya sabemos lo básico sobre este tipo de vulnerabilidad vamos a lanzarnos a la práctica. 

# __IDOR básico__

## __Primer ejemplo__

Empezaremos por un ejemplo muy básico donde vamos a estar explotando un IDOR basado en una API para filtrar direcciones IP privadas. Este ejemplo lo saqué de un blog el cual si le quereis echar un vistazo podeis pinchar en el siguiente [enlace](https://www.zapstiko.com/).

La persona que escribió dicho post nos cuenta que tiene una mala costumbre de activar la intercepción del Burp y ver cada solicitud que el navegador envía al servidor mientras navega por una aplicación web.

Gracias a esto encontró una solicitud API que intentaba obtener los registros de acceso de los usuarios cada vez que visitaba la página.

![555](https://user-images.githubusercontent.com/88755387/130352771-3e3f473b-033a-4342-baa1-349f732a5857.png)

Si nos acercamos al POST data, vemos que tiene un `userId`.

![idor](https://user-images.githubusercontent.com/88755387/130352780-2bb38a7c-9d15-49fa-92f7-6d20caad8711.png)

Luego envió la solicitud al Intruder y bruteforceó el `UserId`, esto le permitió obtener los registros de acceso de 6000 empresas.

Reportó el bug y en las 48h siguientes ya estaba parcheado por la empresa. Podeis comprobar como algo tan simple le dió a esta persona un bounty de 4 cifras :satisfied:.

## __Segundo ejemplo__

En este caso vamos a cambiar la review y el comentario de un user. Estamos utilizando la aplicación vulnerable `juice shop` a través de la cual demostraremos este ataque. A continuación, podemos ver que la review del usuario `admin` es algo que reemplazaremos por malas críticas.

![22-1](https://user-images.githubusercontent.com/88755387/130352788-4262f21b-a946-4207-9bc7-7ea6186c9f23.png)

Necesitamos un `UserID` del primer revisor para el que hemos hecho clic en el botón de :thumbsup:. 

Capturamos la request con Burpsuite:

![23-1](https://user-images.githubusercontent.com/88755387/130352810-5d172bca-3593-422a-8435-45fdabd58112.png)

Cuando verificamos los métodos HTTP disponibles para la confirmación, nos damos cuenta que el método `HTTP PATCH` está habilitado.
 
`El método HTTP PATCH se puede utilizar cuando es necesario actualizar un recurso. Este método es especialmente útil si un recurso es grande y los cambios que se realizan son pequeños.`

![24-1](https://user-images.githubusercontent.com/88755387/130352816-a17d506c-e6e8-4e28-a8ba-82bc98cf369a.png)

Ahora que sabemos que podemos usar `PATCH` vamos a cambiar el mensaje que pone el usuario admin por el nuestro, comprobamos que se refleja en la respuesta de Burpsuite:

![25-1](https://user-images.githubusercontent.com/88755387/130352820-b19ec9b2-8d0f-4775-9550-5fd760bcfbe0.png)

Solo nos falta comprobar que efectivamente este mensaje ha cambiado y está visible para todo el mundo.

![26-2](https://user-images.githubusercontent.com/88755387/130352824-288ffe00-a054-4873-98a4-aa037e032d3e.png)

Ahora la cuestión es, como podriamos mitigar esto? :grimacing:

- Se debe verificar la entrada del usuario antes de usarlo.
- La validación de parámetros debe implementarse correctamente.
- La aplicación debe realizar una Syntactic Validation para verificar las entradas sospechosas.

# __IDOR chain__

## __Explotando un Self Stored XSS con un IDOR__

En este caso no voy a poder explicarlo de una manera práctica debido a que lo saqué de un blog el cual puedes ir directamente pinchando [aquí](https://footstep.ninja/posts/exploiting-self-xss/).

El target se trata de una web que ayuda en la administración de finanzas a diversas empresas. 

En su interior contiene una función para solicitar proveedores en la aplicación el cual este bug hounter aprovecharía para inyectar un payload XSS en el `Name of supplier` y solicitar al proveedor.
No pasó nada hasta que optó por eliminalo y apareció una ventana emergente! 

Posteriormente encontró un IDOR en una request el cual le habilitaba poder ver, editar y eliminar proveedores de otros usuarios.

El cuerpo de la PUT request tenía el siguiente aspecto:

`{"shop_account_request":{"request_name":"Name of supplier","reference_contact_name":"ContactName","reference_contact_email":"Email","reference_contact_phone":"Phone","id":IncrementalID}}`

Esto le permitió establecer un XSS en el parámetro `Name of supplier` para cualquiera de las solicitudes de los proveedores a través de los `IncrementalID`.

Siempre que estos usuarios sintieran que un nombre es extraño o que no debería de estar ahí por razones que ellos desconocen y opten por eliminarlo, se ejecutará el payload XSS y se quedará almacenado en la página web.

# __PortSwigger Labs__

En la plataforma de PortSwigger existen varios labs el cual recomiendo hacerlos una y otra vez para practicar este tipo de vulnerabilidad. En este caso solo voy a comentar uno de los 13 disponibles.

[Access control vulnerabilities and privilege escalation](https://portswigger.net/web-security/access-control)

## __URL-based access control can be circumvented__

Este sitio web tiene un login panel no autenticado en /admin, pero se ha configurado un sistema de front-end para bloquear el acceso externo a esa ruta. Sin embargo, la aplicación de back-end se basa en un marco que admite el encabezado X-Original-URL.

Para resolver el laboratorio, acceda al panel de administración y elimine el usuario carlos.

Solución: Capturamos la request del directorio /admin y la mandamos al Repeater. Una vez ahí añadimos el header `X-Original-URL: /<dirección inválida>` y cambiamos la URL principal a `/`.

![maikyperro](https://user-images.githubusercontent.com/88755387/130352837-924f0ba4-4ead-42a6-bb96-0212121b0848.png)

Observamos que la aplicación devuelve la respuesta "not found". Esto indica que el sistema back-end está procesando la URL del encabezado X-Original-URL.

Ahora cambiamos la cabecera a /admin para ver si nos deja entrar y efectivamente podemos visualizar su contenido.

Para eliminar al usuario Carlos tendríamos que cambiar el header a `/admin/delete` y poner en la URL principal `/?username=carlos`.

## __Cómo prevenir las vulnerabilidades de Access control__

Las vulnerabilidades de `Access control` generalmente se pueden prevenir adoptando un enfoque de defensa en profundidad y aplicando los siguientes principios:

- Nunca confiar únicamente en la ofuscación para el control de acceso.
- A menos que un recurso esté destinado a ser de acceso público, denegar el acceso de forma predeterminada.
- Siempre que sea posible, utilizar un único mecanismo en toda la aplicación para hacer cumplir los controles de acceso.
- A nivel de código, hacer que sea obligatorio para los desarrolladores declarar el acceso permitido para cada recurso y denegar el acceso de forma predeterminada.
- Auditar y proba minuciosamente los controles de acceso para asegurarse de que funcionan según lo diseñado.












