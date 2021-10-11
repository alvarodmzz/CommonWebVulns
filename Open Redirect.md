![image](https://user-images.githubusercontent.com/88755387/136052006-ac316302-048b-409a-83ad-b9d771a15777.png)

## __Summary__

- [__What is a Redirect?__](#What-is-Redirect?)
- [__What is an Open Redirect?__](#What-is-an-Open-Redirect?)
    - [__Classic scenario__](#Classic-scenario)
    - [__Exploitation__](#Exploitation)
    - [__Bad solution__](#Bad-solution)
- [__Mitigations__](#Mitigations)

# __What is a Redirect?__

Una redirección ocurre cuando el sitio web o la aplicación web cambia la URL a la que se accede en el cliente, generalmente externa, aunque también existen los internal redirects el cual se denominan reenvíos o forwards. Hay varias formas de hacer esto desde el back-end. Por lo general, las redirecciones se realizan enviando encabezados HTTP específicos al cliente, pero también podemos crear redirects, por ejemplo, usando código JavaScript.

# __What is an Open Redirect?__

Una redirección abierta (Open Redirect) sucede cuando una aplicación o servidor web utiliza un enlace no validado enviado por el usuario para redirigirlo a un sitio o página web determinada.

A continuación os dejo varios ejemplos de redirects seguros e inseguros:

- Si el sitio web legítimo redirige al cliente a una URL fija, es una redirección segura.
- Si el sitio web legítimo construye de forma segura la URL de redireccionamiento según los parámetros proporcionados por el usuario, es una redirección segura.
- Si el sitio web legítimo construye la URL de redireccionamiento en función de los parámetros proporcionados por el usuario, pero no valida ni filtra lo suficiente la entrada, se trata de una redirección insegura (el atacante puede manipular la entrada).
- Si el sitio web legítimo permite al usuario especificar la URL de redireccionamiento de destino, se trata de una redirección insegura (open redirect).

Por ejemplo, si nuestro dominio es `test.com`, el atacante puede crear la siguiente URL:

```
https://test.com/redirect.php?url=http://attacker.com
```

El atacante puede enviar esta URL como parte de un intento de phishing para redirigir a la víctima a un sitio web malicioso. El atacante esperaría que `test.com` al principio tenga una apariencia confiable y los haga caer en la estafa de phishing.

El siguiente código PHP crea un open redirect:

```php
$redirect = $_GET['url'];  header("Location: " . $redirect);
```

Esto es debido a que el atacante puede proporcionar una URL del sitio web malicioso en el valor del parámetro url de la solicitud GET y esta URL de destino se enviará como el `Location` header, redirigiendo al cliente a la página del atacante.

## __Classic scenario__

La siguiente imagen muestra un sitio web vulnerable a open redirect llamado `victim-site.com`. En esta ocasión tenemos una página de inicio de sesión que redirigirá al usuario a la página especificada en el parámetro `returnURL` después de iniciar sesión correctamente.

![image](https://user-images.githubusercontent.com/88755387/136060350-643b2cb3-a681-4d46-ae02-b994bdca280b.png)

En el siguiente fragmento podemos ver el código vulnerable el cual recibe la página de redirección del cliente en el parámetro `returnURL`, y lo establece directamente en el encabezado `Location`:

```php
# [...]

if ($count == 1)
{
    $_SESSION['loggedIn'] = "true";
    header("Location: index.php");

    if ($_GET["returnURL"] != NULL)
    {
        echo header( 'Location: ' . $_GET["returnURL"] ); # VULNERABLE CODE HERE
    }
} 

# [...]
```

La siguiente imagen muestra la solicitud del cliente y la respuesta del servidor. Es posible ver que el valor del parámetro `returnURL` en la solicitud es el mismo que en el `Location` header de la respuesta:

![image](https://user-images.githubusercontent.com/88755387/136061246-8b3053c2-a309-4eeb-b913-925a7f141889.png)

## __Exploitation__

Un atacante puede aprovechar esta vulnerabilidad para redirigir a la víctima a un sitio web de phishing malicioso, simplemente enviándole la siguiente URL:

```
http://victim-site.com/login.php?returnUrl=http://attacker-site.com
```

Para ofuscar la página maliciosa, el atacante puede utilizar una versión encodeada en formato URL:

```
%68%74%74%70%3a%2f%2f%61%74%74%61%63%6b%65%72%2d%73%69%74%65%2e%63%6f%6d (http://attacker-site.com)
```

El sitio web de phishing engañará a la víctima y la atraerá para que vuelva a insertar sus credenciales y las envie directamente al atacante.

## __Bad solution__

En muchas ocasiones, la solución para prevenir este tipo de ataques va en contra de las mejores prácticas y no mitiga completamente el problema.

El siguiente código muestra una mala práctica para corregir esta vulnerabilidad:

```php
# [...]

if ($count == 1)
{
    $_SESSION['loggedIn'] = "true";
    header("Location: index.php");

    if ($_GET["returnURL"] != NULL)
    {
        echo header( 'Location: htt://victim-site.com' . $_GET["returnURL"] ); # VULNERABLE CODE HERE
    }
} 

# [...]
```

El desarrollador asumió que, dado que la aplicación contiene una cadena prefijada en el `Location` header, los atacantes no podrán afectar dicho dominio. Este punto de vista es bastante lógico a primera vista y, como tal, esto es lo que sucede al intentar explotarlo:

![image](https://user-images.githubusercontent.com/88755387/136063476-99ae4187-e7eb-4cab-b2e0-8b29bc47c9b9.png)

Podemos ver que dicho header está redirigiendo a una dirección inválida que incluye tanto el dominio del prefijo como la URL enviada en el parámetro `returnURL`. Por lo tanto, obtenemos:

![image](https://user-images.githubusercontent.com/88755387/136069839-0dabef05-ce6d-4286-89a5-1fb2dfb9dd00.png)

Esto lo podemos bypassear de una forma bastante simple añadiendo un simple punto `.` en el inicio de la URL:

```
http://victim-site.com/login.php?returnUrl=%2Fattacker-site.com
```

La dirección de `Location` quedaría de la siguiente manera: `http://victim-site.com.attacker-site.com` y el atacante solo necesitará crear su sitio de phishing bajo el subdominio victim-site.com en lugar de www.

# __Mitigations__

La forma más fácil y eficaz de prevenir los open redirects vulnerables sería no permitir que el usuario pueda controlar a dónde lo redirecciona dicha página. Si tenemos que redirigir al usuario en función de las URLs, siempre debemos usar un ID que se resuelva internamente en la URL respectiva.

En el caso de que queramos que el usuario pueda emitir redirecciones, debemos usar una página de redireccionamiento que requiera que el usuario haga clic en el enlace en lugar de simplemente redirigirlos. También debemos verificar que la URL comience con `http://` o `https://` y también invalidar todas las demás URL para evitar el uso de URI maliciosas como `javascript :`.








