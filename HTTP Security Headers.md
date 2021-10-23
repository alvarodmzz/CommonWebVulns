![image](https://user-images.githubusercontent.com/88755387/138565989-f0fa4c6a-273d-48e7-989c-3121cabcc7e5.png)

## __Summary__

- [__Cabeceras HTTP__](#Cabeceras-HTTP)
- [__¿Qué son las cabeceras de seguridad HTTP?__](#¿Qué-son-las-cabeceras-de-seguridad-HTTP?)
- [__¿Cómo mejoran la seguridad de las aplicaciones web?__](#¿Cómo-mejoran-la-seguridad-de-las-aplicaciones-web?)
- [__Las cabeceras de seguridad HTTP más importantes__](#Las-cabeceras-de-seguridad-HTTP-más-importantes)
    - [__Strict-Transport-Security__](#Strict-Transport-Security)
    - [__Content-Security-Policy__](#Content-Security-Policy)
    - [__X-Frame-Options__](#X-Frame-Options)
- [__Otras cabeceras de seguridad HTTP útiles__](#Otras-cabeceras-de-seguridad-HTTP-útiles)
    - [__Expect-CT__](#Expect-CT)
    - [__X-Content-Type-Options__](#X-Content-Type-Options)
    - [__Fetch Metadata Headers__](#Fetch-Metadata-Headers)
- [__Resumen__](#Resumen)

# __Cabeceras HTTP__

Para poder entender las cabeceras de seguridad primero vamos a ver una breve explicación de lo que son las cabeceras HTTP:

Las cabeceras (en inglés headers) HTTP permiten al cliente y al servidor enviar información adicional junto a una petición o respuesta. Una cabecera de petición esta compuesta por su nombre (no sensible a las mayúsculas) seguido de dos puntos `:`, y a continuación su valor (sin saltos de línea). Los espacios en blanco a la izquierda del valor son ignorados.

# __¿Qué son las cabeceras de seguridad HTTP?__

Las cabeceras de seguridad HTTP son un subconjunto de las cabeceras HTTP el cual se intercambian entre un cliente web (normalmente un navegador) y un servidor para especificar los detalles relacionados con la seguridad de la comunicación HTTP. Algunas cabeceras HTTP que están indirectamente relacionadas con la privacidad y la seguridad también pueden considerarse cabeceras de seguridad HTTP. Al habilitar las cabeceras adecuadas en las aplicaciones web y en la configuración del servidor web, podemos mejorar la resistencia de la aplicación web contra muchos ataques comunes, incluyendo el cross-site scripting (XSS) y el clickjacking.

# __¿Cómo mejoran la seguridad de las aplicaciones web?__

Las cabeceras de seguridad HTTP proporcionan una capa adicional de seguridad al restringir los comportamientos que el navegador y el servidor permiten una vez que la aplicación web se está ejecutando. En muchos casos, la implementación de las cabeceras adecuadas es un aspecto crucial de la configuración de una aplicación con buenas prácticas, pero ¿cómo saber cuáles utilizar?

Al igual que ocurre con otras tecnologías web, las cabeceras HTTP aparecen y desaparecen en función del soporte del proveedor del navegador. Especialmente en el campo de la seguridad, las cabeceras que eran ampliamente soportadas hace unos años pueden quedar ya obsoletas. Al mismo tiempo, las propuestas completamente nuevas pueden obtener apoyo universal en cuestión de meses. Estar al día de todos estos cambios no es fácil.

# __Las cabeceras de seguridad HTTP más importantes__

Veamos un resumen de las cabeceras seleccionadas, empezando por algunas de las cabeceras de respuesta HTTP más conocidas.

## __Strict-Transport-Security__

Cuando se activa en el servidor, HTTP Strict Transport Security (HSTS) impone el uso de conexiones HTTPS cifradas en lugar de la comunicación HTTP en texto plano. Una cabecera HSTS típica podría ser:

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

Esto informaría al navegador web visitante de que el sitio actual (incluidos los subdominios) es sólo HTTPS y el navegador debería acceder a él a través de HTTPS durante los próximos 2 años (el valor de edad máxima en segundos). La `preload directive` indica que el sitio está presente en una lista global de sitios sólo HTTPS. La preload tiene como objetivo acelerar la carga de las páginas y eliminar el riesgo de ataques de tipo man-in-the-middle (MITM) cuando se visita un sitio por primera vez.

## __Content-Security-Policy__

La cabecera Content Security Policy (CSP) es la navaja suiza de las cabeceras de seguridad HTTP y la forma recomendada de proteger sus sitios web y aplicaciones contra los ataques XSS. Permite controlar con precisión las fuentes de contenido permitidas y muchos otros parámetros. Una cabecera CSP básica para permitir sólo los activos del origen local es:

```
Content-Security-Policy: default-src 'self'
```

Otras directivas incluyen `script-src`, `style-src` e `img-src` para especificar las fuentes permitidas para los scripts, las hojas de estilo CSS y las imágenes. Por ejemplo, si se especifica` script-src 'self'` sólo se permitirán scripts del origen local. También puedes restringir las fuentes de los plugins usando `plugin-types` (no soportado por Firefox) u `object-src`.

## __X-Frame-Options__

Esta cabecera se introdujo por primera vez en Microsoft Internet Explorer para proporcionar protección contra los ataques XSS que implican iframes HTML. Para evitar que la página actual se cargue en cualquier iframe, se utilizaría:

```
X-Frame-Options: deny
```

Otros valores soportados son `sameorigin` para permitir la carga en iframes con el mismo origen y `allow-from` para indicar URLs específicas. Esta cabecera puede sustituirse normalmente por directivas CSP adecuadas.

# __Otras cabeceras de seguridad HTTP útiles__

Aunque no son tan cruciales como CSP y HSTS, las siguientes cabeceras también pueden ayudar a endurecer la aplicación web.

## __Expect-CT__

Para evitar la falsificación de certificados de sitios web, se puede utilizar la cabecera Expect-CT para indicar que sólo se deben aceptar los nuevos certificados añadidos a los registros de Certificate Transparency. Una cabecera típica sería:

```
Expect-CT: max-age=86400, enforce, 
    report-uri="https://domain.com/report"
```

Con la directiva `enforce`, se indica a los clientes que rechacen las conexiones que infrinjan la política de transparencia de certificados. La directiva opcional `report-uri` indica una ubicación para informar de los fallos.

## __X-Content-Type-Options__

Cuando está presente en las respuestas del servidor, esta cabecera obliga a los navegadores web a seguir estrictamente los tipos MIME especificados en las cabeceras `Content-Type`. Esto protege a los sitios web de los ataques XSS que abusan de las capacidades de rastreo de MIME para suministrar código malicioso enmascarado como un tipo MIME no ejecutable. La cabecera sólo tiene una directiva:

```
X-Content-Type-Options: nosniff
```

## __Fetch Metadata Headers__

Un nuevo conjunto de cabeceras del lado del cliente permite al navegador informar al servidor sobre diferentes atributos de la solicitud HTTP. Actualmente existen cuatro cabeceras:

- `Sec-Fetch-Site`: Indica la relación prevista entre el origen y el destino.
- `Sec-Fetch-Mode`: Indica el modo de solicitud previsto.
- `Sec-Fetch-User`: Indica si la solicitud fue iniciada por el usuario.
- `Sec-Fetch-Dest`: Indica el destino de la solicitud.

Si el servidor y el navegador lo admiten, estas cabeceras pueden utilizarse para informar al servidor sobre los comportamientos previstos de la aplicación para identificar solicitudes sospechosas.

# __Resumen__

Las cabeceras de seguridad HTTP son a menudo una forma fácil de mejorar la seguridad de las aplicaciones web sin cambiar la propia aplicación, por lo que siempre es una buena idea utilizar las cabeceras más actuales. Sin embargo, debido a que el soporte de los proveedores para las cabeceras HTTP puede cambiar tan rápidamente, es difícil mantener todo actualizado, especialmente si se está trabajando con cientos de sitios web. 








