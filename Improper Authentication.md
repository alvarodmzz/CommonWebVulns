![oauth](https://user-images.githubusercontent.com/88755387/133160635-4f3177db-8257-4230-9e94-e4c307f037bd.jpg)

## __Summary__

- [__Descripción__](#Descripción)
- [__Práctica__](#Práctica)
- [__Tipos de I.A.__](#Tipos-de-I.A.)
  - [__SAML__](#SAML)
  - [__Oauth__](#Oauth)
    - [__Forced OAuth profile linking__](#Forced-OAuth-profile-linking)
  - [__SSO__](#SSO)
- [__Un caso curioso__](#Un-caso-curioso)

# __Descripción__

La autenticación incorrecta ocurre cuando una aplicación verifica incorrectamente la identidad de un usuario.

Un software valida incorrectamente la información de inicio de sesión del usuario y, como resultado, un atacante puede obtener ciertos privilegios dentro de la aplicación o revelar información confidencial que le permite acceder a datos confidenciales y provocar la ejecución de código arbitrario.

La debilidad se introduce durante las etapas de Arquitectura, Diseño e Implementación.

# __Práctica__

A continuación os voy a dejar dos ejemplos que ayudan a ilustrar la naturaleza de esta debilidad y describen métodos o técnicas que pueden usarse para mitigar el riesgo.

## __Example 1__

El siguiente código intenta garantizar que el usuario ya haya iniciado sesión. De lo contrario, el código realiza la autenticación con el nombre de usuario y contraseña proporcionados por el usuario. 

Si tiene éxito, configura las cookies de inicio de sesión y de usuario para `recordar` que este ya ha iniciado sesión. Finalmente, el código realiza tareas de administrador si el usuario que inició sesión tiene el nombre `Administrator`, como se registra en la cookie de usuario.

```perl
my $q = new CGI;
if ($q->cookie('loggedin') ne "true") {
  if (! AuthenticateUser($q->param('username'), $q->param('password'))) {
    ExitError("Error: you need to log in first");
  }
  else {
    # Set loggedin and user cookies
    $q->cookie(
      -name => 'loggedin',
      -value => 'true'
      );
    $q->cookie(
      -name => 'user',
      -value => $q->param('username')
      );
  }
}
if ($q->cookie('user') eq "Administrator") {
  DoAdministratorTasks();
}
```

Desafortunadamente, este código puede ser bypasseado. El atacante puede configurar las cookies de forma independiente para que el código no verifique el nombre de usuario y la contraseña. El atacante podría hacer esto con una solicitud HTTP que contenga encabezados como:

```
GET /cgi-bin/vulnerable.cgi HTTP/1.1
Cookie: user=Administrator
Cookie: loggedin=true

[body of request]
```

Al configurar la cookie de inicio de sesión en `true`, el atacante bypassea toda la verificación de autenticación. Al utilizar el valor `Administrator` en la cookie del usuario, el atacante también obtiene privilegios para administrar el software.

## __Example 2__

En enero de 2009, un atacante pudo obtener acceso de administrador a un servidor de Twitter porque el servidor no restringió el número de intentos de inicio de sesión. El atacante apuntó a un miembro del equipo de soporte de Twitter y pudo adivinar con éxito la contraseña del miembro mediante un ataque de fuerza bruta adivinando una gran cantidad de palabras comunes. Después de obtener acceso como miembro del personal de apoyo, el atacante usó el admin panel para obtener acceso a 33 cuentas que pertenecían a celebridades y políticos. Al final, se enviaron mensajes de Twitter falsos que parecían provenir de las cuentas comprometidas.

# __Tipos de I.A.__

A la hora de encontrar este tipo de vulnerabilidad podemos hablar de tres principales tipos de flujos de autenticación:

- SAML (Security Assertion Markup Language)
- Oauth (Open Authorization)
- SSO (sistema Single Sign-On)

## __SAML__

SAML (Security Assertion Markup Language) es un estándar de código abierto basado en XML que permite el intercambio de información, tanto de autenticación como de autorización entre diferentes partes: un identity provider (proveedor de identidad) y un service provider (proveedor de servicios).

## __Oauth__

Cuando hablamos de OAuth (Open Authorization), nos referimos a una solución de administración de identidad y acceso (IAM). Su finalidad es otorgar autorizaciones a los usuarios. Se trata de un protocolo para pasar la autorización de un servicio a otro sin compartir las credenciales de usuario reales, como un nombre de usuario y contraseña. Con esta herramienta, un usuario puede iniciar sesión en una plataforma y luego estar autorizado para realizar acciones y ver datos en otra plataforma.

### __Forced OAuth profile linking__

Este lab nos brinda la opción de adjuntar un perfil de redes sociales a nuestra cuenta para que podamos iniciar sesión a través de OAuth en lugar de usar el nombre de usuario y la contraseña normales. Debido a la implementación insegura del flujo de OAuth por parte de la aplicación cliente, un atacante puede manipular esta funcionalidad para obtener acceso a las cuentas de otros usuarios.

El primer paso es hacer el flujo normal como lo haría un usuario a la hora de loguearse en una página y adjuntar sus redes sociales para a si en un futuro poder entrar a través de ellas sin necesidad de pasar por un panel de login.

El segundo paso es activar el Burp para comprender como viaja la opción "Attach a social profile".

![image](https://user-images.githubusercontent.com/88755387/133300768-271f86fb-30e7-46a0-a886-bf7592f6605d.png)

En la primera request observamos que `redirect_uri`, para esta funcionalidad, envía el código de autorización a `/oauth-linked`. También es importante destacar que la solicitud no incluye un parámetro de estado para protegerse contra ataques CSRF. 

![image](https://user-images.githubusercontent.com/88755387/133301055-488f3491-8e2c-42dd-8565-edb60bd7eec8.png)


En la segunda request copiamos la URL y vamos a dropearla para garantizar que el código no se utilice y, por lo tanto, siga siendo válido.

![image](https://user-images.githubusercontent.com/88755387/133301586-58f7f06c-9ee6-4a53-a316-6958d3d38d64.png)

El siguiente paso es desloguearnos y entrar en el apartado:

![image](https://user-images.githubusercontent.com/88755387/133302044-33d325f2-96bf-49a4-9bbe-c109a15d862b.png)

Una vez dentro vamos a cambiar el cuerpo de la respuesta. Vamos a crear un iframe en el que el atributo `src` apunte a la URL que acabamos de copiar.

> Se utiliza un iframe o inline frame para mostrar objetos externos, incluidas otras páginas web dentro de una página web. Un iframe actúa prácticamente como un mini navegador web dentro de un navegador web. Además, el contenido dentro de un iframe existe completamente independiente de los elementos circundantes.

La sintaxis básica para agregar un iframe a una página web es la siguiente:
```
<iframe src="URL"></iframe>
```

Ahora que ya sabemos como utilizar `iframe` vamos a crear nuestro payload:

```
<iframe src="https://acb81f441e2fd00e80603ca8008d005d.web-security-academy.net/oauth-linking?code=YLMhJAFFAVxZih7BjbHvGKI4isRsTono6lqTbU0wStR"></iframe>
```

Una vez que tenemos todo listo vamos a mandárselo a la víctima para efectuar nuestro CSRF con éxito. Cuando su navegador cargue el iframe, completará el flujo de OAuth usando su perfil de redes sociales, adjuntándolo a la cuenta de administrador en el sitio web del blog.

![image](https://user-images.githubusercontent.com/88755387/133303216-84047e2b-5a5b-490e-8f36-cbb3b1645b18.png)

El último paso es loguearnos a través de nuestras redes sociales.

![image](https://user-images.githubusercontent.com/88755387/133303688-f1862e7f-ec86-4a6d-8467-3cc035a3dde1.png)

Y ya seríamos Admin:

![image](https://user-images.githubusercontent.com/88755387/133304000-37d410c1-e48f-442c-bc57-f03633606455.png)

## __SSO__

El inicio de sesión único (SSO) es una capacidad de autenticación que permite que los usuarios accedan a varias aplicaciones con un único conjunto de credenciales de inicio de sesión. Por lo general, las empresas utilizan SSO para proporcionar un acceso más sencillo a una serie de aplicaciones web, en las instalaciones y en la nube para obtener una mejor experiencia del usuario.

# __Un caso curioso__

Servicio de inicio de sesión único (SSO) basado en SAML proporcionado por partners:

Google ofrece un servicio de inicio de sesión único (SSO) basado en SAML con el que los partners tienen un control completo sobre la autorización y la autenticación de las cuentas de usuario alojadas que pueden acceder a aplicaciones web, como Gmail o Google Calendar. 

En el modelo SAML, Google actúa como proveedor de servicios y proporciona servicios como Gmail y páginas de inicio. Los partners de Google actúan como proveedores de identidades y controlan los nombres de usuario, las contraseñas y otra información utilizada para identificar, autenticar y autorizar a los usuarios en aplicaciones web alojadas por Google.








