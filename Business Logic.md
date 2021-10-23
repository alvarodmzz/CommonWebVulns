![image](https://user-images.githubusercontent.com/88755387/138563553-2d3b067a-9f60-407c-9dd1-d8e15441934c.png)

## __Summary__

- [__¿Qué son las vulnerabilidades business logic?__](#¿Qué-son-las-vulnerabilidades-business-logic?)
- [__¿Dónde se encuentran?__](#¿Dónde-se-encuentran?)
- [__PortSwigger Labs__](#PortSwigger-Labs)
    - [__Confianza excesiva en los controles del lado del cliente__](#Confianza-excesiva-en-los-controles-del-lado-del-cliente)
    - [__Password Reset Broken Logic__](#Password-Reset-Broken-Logic)
    - [__Low-Level Logic Flaw__](#Low-Level-Logic-Flaw)
- [__Mitigaciones__](#Mitigaciones)

# __¿Qué son las vulnerabilidades business logic?__

Las vulnerabilidades de business logic son fallos en el diseño y la implementación de una aplicación que permiten a un atacante provocar un comportamiento no deseado. Esto permite potencialmente a los atacantes manipular la funcionalidad legítima para lograr un objetivo malicioso. Estas fallas son generalmente el resultado de no anticipar los estados inusuales de la aplicación que pueden ocurrir y, en consecuencia, no manejarlos con seguridad.

```
En este contexto, el término "business logic" se refiere simplemente al conjunto de reglas que definen el funcionamiento de la aplicación. Como estas reglas no siempre están directamente relacionadas con un negocio, las vulnerabilidades asociadas también se conocen como "application logic vulnerabilities" o simplemente "logic flaws".
```

Los fallos lógicos suelen ser invisibles para las personas que no los buscan explícitamente, ya que normalmente no quedan expuestos por el uso normal de la aplicación. Sin embargo, un atacante puede ser capaz de explotar las peculiaridades de comportamiento al interactuar con la aplicación de maneras que los desarrolladores nunca pretendieron.

Uno de los principales propósitos de la lógica de negocio es hacer cumplir las reglas y restricciones que se definieron al diseñar la aplicación o la funcionalidad. En términos generales, las reglas de negocio dictan cómo debe reaccionar la aplicación cuando se produce un determinado escenario. Esto incluye evitar que los usuarios hagan cosas que tengan un impacto negativo en el negocio o que simplemente no tengan sentido.

Los fallos en la lógica pueden permitir a los atacantes eludir estas reglas. Por ejemplo, podrían completar una transacción sin pasar por el flujo de trabajo de compra previsto. En otros casos, la validación de los datos suministrados por el usuario, que no es correcta o no existe, puede permitir a los usuarios realizar cambios arbitrarios en los valores críticos de la transacción o enviar datos sin sentido. Al pasar valores inesperados a la lógica del lado del servidor, un atacante puede inducir a la aplicación a hacer algo que no se supone que deba hacer.

Las vulnerabilidades logic-based pueden ser extremadamente diversas y a menudo son únicas para la aplicación y su funcionalidad específica. Identificarlas suele requerir un cierto conocimiento humano, como la comprensión del dominio empresarial o los objetivos que podría tener un atacante en un contexto determinado. Esto hace que sean difíciles de detectar mediante escáneres de vulnerabilidad automatizados. Como resultado, los fallos lógicos son un gran objetivo para los bug-bounty hunters y los manual testers en general.

# __¿Dónde se encuentran?__

Los fallos lógicos son especialmente comunes en sistemas demasiado complicados que ni siquiera el propio equipo de desarrollo entiende del todo. Para evitar los fallos lógicos, los desarrolladores deben entender la aplicación en su conjunto. Esto incluye ser consciente de cómo las diferentes funciones pueden combinarse de forma inesperada. Los desarrolladores que trabajan en grandes bases de código pueden no tener un conocimiento profundo de cómo funcionan todas las áreas de la aplicación. Alguien que trabaje en un componente podría hacer suposiciones erróneas sobre el funcionamiento de otro componente y, como resultado, introducir inadvertidamente graves fallos lógicos. Si los desarrolladores no documentan explícitamente las suposiciones que se hacen, es fácil que este tipo de vulnerabilidades se cuelen en una aplicación.

Los siguientes vectores de ataque son los comunes:

- Indicadores de autenticación y escalada de privilegios.
- Manipulación de parámetros críticos y acceso a información/contenido no autorizado.
- Manipulación de cookies por parte del desarrollador y business process/logic bypasss.
- Identificación de parámetros LDAP y acceso a infraestructuras críticas.
- Explotación de las limitaciones empresariales.
- Business flow bypass.
- Explotación de rutinas de negocio del lado del cliente incrustadas en JavaScript, Flash o Silverlight.
- Extracción de la identidad o del perfil.
- Acceso a archivos o URLs no autorizadas y extracción de información comercial.
- Denegación de servicios (DoS) con business logic.


# __PortSwigger Labs__

A continuación vamos a tratar varios ejemplos el cual están sacados de [PortSwigger](https://portswigger.net/web-security/logic-flaws/examples). Al no haber mucha información a cerca de este tipo de vulnerabilidad, casi todo lo que acabo de explicar pertenece a dicha página.

## __Confianza excesiva en los controles del lado del cliente__

En este ejemplo, tenemos un eCommerce website que vende una chaqueta por más de 1.000 dólares. Cuando el artículo se añade al carrito, el precio también se envía en la solicitud. Podemos interceptar esta solicitud y ajustar el precio del artículo nosotros mismos utilizando Burp
Suite.

![image](https://user-images.githubusercontent.com/88755387/138562335-4ec9def3-c73a-4bfc-b48d-7e6f1ff8474c.png)

Si cambiamos el precio a 1, no habría ningún problema y la podríamos adquirir por dicho precio.

![image](https://user-images.githubusercontent.com/88755387/138562354-1b348732-3170-40bf-bedc-853d23de0755.png)

 # __Password Reset Broken Logic__

 En este ejemplo, podemos solicitar un restablecimiento de contraseña para nuestra propia cuenta para generar un token de restablecimiento válido. Sin embargo, la aplicación web no parece realizar comprobaciones con el token para asegurar su validez. Encontramos que cuando el token es eliminado, somos capaces de restablecer una contraseña para cualquier cuenta de usuario actualizando el parámetro `username`.

 ![image](https://user-images.githubusercontent.com/88755387/138562526-7be78672-8c48-4348-86b0-6282bae87cc0.png)

 Con los valores de los tokens eliminados, podemos entonces ajustar el parámetro de nombre de usuario para reflejar la cuenta que deseamos restablecer. Esto nos permite ahora iniciar sesión usando `carlos:password` como nuestras credenciales.

 ![image](https://user-images.githubusercontent.com/88755387/138562571-a34f8dd1-444c-418b-bdd8-644a529b244d.png)

## __Low-Level Logic Flaw__

En este ejemplo, estamos trabajando con el mismo eCommerce website. Sin embargo, nuestro ataque anterior no funciona. Esta vez, vamos a intentar añadir tantos artículos al carrito para que el `Precio total` se desborde. La mayoría de los lenguajes de programación no pueden contener enteros de más de 2,147b de valor. En este caso particular, añadir suficientes artículos al carrito hará que el precio total se desborde y comience a incrementarse en valores negativos.

Burp Suite puede realizar esto fácilmente por nosotros. Si interceptamos una petición que está añadiendo 99 artículos al carrito, podemos enviarla al Intruder. En lugar de especificar un payload, los borraremos y estableceremos nuestro tipo de payload a un "Null payload". Esto esencialmente nos permitirá repetir nuestra misma solicitud en masa un cierto, o infinito, número de veces.

![image](https://user-images.githubusercontent.com/88755387/138563031-0b586870-c600-4a5d-9bb1-7003fb34934a.png)

Podemos ver que al hacer esto, ahora hemos añadido una tonelada de artículos al carrito, y provocamos que el precio total devuelva un valor negativo.

![image](https://user-images.githubusercontent.com/88755387/138563076-d282d80b-bc75-4174-aef2-5e4b0fc2d103.png)

Ahora sólo tenemos que modificar el carrito de la manera correcta para que el precio total se ajuste a un valor entre 0 y 100 dólares, ya que tenemos crédito de la tienda de esa cantidad. En ese momento, podemos realizar el pedido.

![image](https://user-images.githubusercontent.com/88755387/138563123-9e2caf49-46d1-4217-82cf-e32d3c1c1f27.png)

# __Mitigaciones__

En resumen, las claves para prevenir las vulnerabilidades de business logic son:

- Asegurarse de que los desarrolladores y los testers entienden el dominio al que sirve la aplicación.
- Evitar hacer suposiciones implícitas sobre el comportamiento del usuario o de otras partes de la aplicación.

Deberiamos identificar qué suposiciones hemos hecho sobre el estado del lado del servidor e implementar la lógica necesaria para verificar que estas suposiciones se cumplen. Esto incluye asegurarse de que el valor de cualquier entrada es sensato antes de proceder.

También es importante asegurarse de que tanto los desarrolladores como los testers son capaces de entender completamente estas suposiciones y cómo se supone que la aplicación debe reaccionar en diferentes escenarios. Esto puede ayudar al equipo a detectar fallos lógicos lo antes posible.

Debido a la naturaleza relativamente única de muchos fallos lógicos, es fácil descartarlos como un error puntual debido a un error humano y seguir adelante. Sin embargo, como hemos demostrado, estos fallos son a menudo el resultado de malas prácticas en las fases iniciales de la construcción de la aplicación. Analizar por qué existía un fallo lógico en primer lugar, y cómo se le pasó por alto al equipo, nos puede ayudar a detectar puntos débiles en nuestros procesos. Haciendo pequeños ajustes, podemos aumentar la probabilidad de que fallos similares se corten de raíz o se detecten antes en el proceso de desarrollo.











