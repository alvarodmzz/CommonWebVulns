![image](https://user-images.githubusercontent.com/88755387/136995469-78ade12b-1603-4ad2-9354-29f6cf90e60b.png)


## __Summary__

- [__¿Qué son?__](#¿Qué-son?)
- [__¿Cómo suceden?__](#¿Cómo-suceden?)
    - [__Durante el aprovisionamiento__](#Durante-el-aprovisionamiento)
    - [__Durante la eliminación__](#Durante-la-eliminación)
- [__¿Cómo los podemos evadir?__](#¿Cómo-los-podemos-evadir?)
- [__Ejemplo__](#Ejemplo)

# __¿Qué son?__

Un subdomain takeover se produce cuando un atacante obtiene el control sobre un subdominio de un dominio de destino. Normalmente, esto sucede cuando el subdominio tiene un nombre canónico (CNAME) en el Domain Name System (DNS), pero ningún host proporciona contenido para él.

Esto puede suceder porque aún no se ha publicado un host virtual o se ha eliminado un host virtual. Un atacante puede hacerse cargo de ese subdominio proporcionando su propio host virtual y, a continuación, alojando su propio contenido para él.

Si un atacante puede hacer esto, potencialmente puede leer las cookies establecidas desde el dominio principal, realizar ataques XSS o eludir las políticas de seguridad de contenido, lo que les permite capturar información protegida (incluidos los inicios de sesión) o enviar contenido malicioso a usuarios desprevenidos.

Un subdominio es como una toma de corriente. Si tenemos nuestro propio dispositivo (host) conectado, todo va bien. Sin embargo, si quitamos el dispositivo de la toma de corriente (o aún no se ha enchufado uno), alguien podría enchufar el suyo. Para solucionar esto debemos cortar la energía en el disyuntor o en la caja de fusibles (DNS) para evitar que otra persona utilice dicho enchufe.

# __¿Cómo suceden?__

Si el proceso de provisioning o deprovisioning (eliminación) de un host virtual no se maneja correctamente, puede haber una oportunidad para que un atacante se apodere de un subdominio.

## __Durante el aprovisionamiento__

Un atacante configura un host virtual para un nombre de subdominio que nosotros compramos en el proveedor de alojamiento, antes de que nosotros lo hagamos.

Supongamos que nosotros controlamos el dominio `ejemplo.com`. Queremos añadir un blog en `blog.ejemplo.com` y decidimos utilizar un proveedor de alojamiento que mantenga una plataforma de blogs. (En lugar de `blog`, podemos sustituirlo por "plataforma de comercio electrónico", "plataforma de atención al cliente" o cualquier otro escenario de alojamiento virtual "basado en la nube"). El proceso que seguimos podría ser el siguiente:

1. Registramos el nombre `blog.ejemplo.com` en un registrador de dominios.
2. Configuramos los registros DNS para dirigir a los navegadores que quieran acceder a `blog.ejemplo.com` para que vayan al host virtual.
3. Creamos un host virtual en el proveedor de alojamiento.

A menos que el proveedor de alojamiento sea muy cuidadoso para verificar que la entidad que configura el host virtual es realmente el propietario del nombre del subdominio, un atacante que sea más rápido que tú podría crear un host virtual con el mismo proveedor de alojamiento, utilizando tu nombre de subdominio. En tal caso, tan pronto como configure el DNS en el paso 2, el atacante podrá alojar contenido en su subdominio.

## __Durante la eliminación__

Retiramos nuestro host virtual, pero un atacante crea uno utilizando el mismo nombre y el mismo proveedor de alojamiento.

Nosotros (o nuestra empresa) decidimos que ya no queremos mantener un blog, así que retiramos el host virtual del proveedor de alojamiento. Sin embargo, si no eliminamos la entrada DNS que apunta al proveedor de alojamiento, un atacante ahora puede crear su propio host virtual con ese proveedor, reclamar nuestro subdominio y alojar su propio contenido bajo ese subdominio.

# __¿Cómo los podemos evadir?__

Si somos el propietario del dominio existen varios pasos que podemos realizar:

1. Revisamos nuestras entradas de DNS y eliminamos todas las entradas que estén activas pero que ya no estén en uso, especialmente aquellas que apuntan a servicios externos. Nos tenemos que asegurar de eliminar el registro CNAME obsoleto en el archivo de zona DNS.
2. Nos aseguramos de que nuestros servicios externos estén configurados para escuchar nuestro wildcard DNS.
3. No nos podemos olvidar del "off-boarding": tenemos que agregar "DNS entry removal" a nuestra checklist.
4. Al crear un nuevo recurso, tenemos que hacer que la creación del registro DNS sea el último paso del proceso para evitar que apunte a un dominio inexistente.
5. Tenemos que supervisar continuamente nuestras entradas de DNS y asegurarnos de que no haya registros de DNS colgando.

# __Ejemplo__

La primera tool que vamos a utilizar se llama `sonar.sh` y puede ser descargada en el siguiente enlace: https://github.com/jakejarvis/subtake/blob/main/sonar.sh.

Esta es una herramienta que descarga el conjunto de datos y genera un archivo de texto simple de CNAME apuntado a cualquiera de los servicios que hayamos elegido, como por ejemplo GitHub o Shopify debido a que son dos de los servicios que mas subdominios abandonados tienen.

Antes de lanzar el script debemos descargar la última dataset de CNAMES de Rapid7, el cual podemos encontrar en la siguiente página: https://opendata.rapid7.com/sonar.fdns_v2/.

De esta manera quedaría nuestro comando:

```
./sonar.sh 2021-09-24-1632523752 sonar_output.txt
```

![image](https://user-images.githubusercontent.com/88755387/137013100-1d993f09-9d5b-4579-bfc0-0a7bae9968c5.png)

Este nuevo archivo de texto contiene subdominios activos y abandonados que apuntan a cualquiera de los servicios enumerados anteriormente, todavía tenemos que reducir el número de subdominios encontrados debido a que NO todos ellos son vulnerables como tal, aquí es donde la tool `subtake` entra en juego. Para instalar subtake, ejecutamos el siguiente comando:

```
go get github.com/jakejarvis/subtake
```

Si no os funciona os descargais el repositorio, haceis un `go build main.go` y ya teneis vuestro subtake listo. 

El siguiente paso es lanzarla para sacar los subdominios vulnerables:

```
subtake -f sonar_output.txt -c fingerprints.json -t 50 -ssl -a -o vulnerable.txt
```

![image](https://user-images.githubusercontent.com/88755387/137014792-264bdbea-b529-4115-b295-806c46543ca9.png)


Este proceso puede tardar varios minutos así que podemos cancelarlo cuando queramos y lo que haya recopilado hasta el momento se quedará guardado en el archivo `vulnerable.txt`.

Una vez que tenemos el output vamos a buscar los subdominios de GitHub, por ejemplo:

```
grep GitHub vulnerable.txt
```

![image](https://user-images.githubusercontent.com/88755387/137022026-c3355be0-43f8-42b6-94fc-9519bf470f12.png)


Ahora que ya hemos elegido un subdominio vamos a seguir los pasos que nos dice el siguiente repositorio de GitHub: https://github.com/EdOverflow/can-i-take-over-xyz/issues/37.

Lo primero es crear un repositorio, luego vamos a la pestaña de `Settings` y una vez ahí dentro elegimos el siguiente Source:

![image](https://user-images.githubusercontent.com/88755387/137022158-48200ed7-f00d-4e6a-9ba4-1cd7a8d2c09a.png)

Y finalmente en esa misma pestaña ponemos nuestro subdominio en el siguiente apartado:

![image](https://user-images.githubusercontent.com/88755387/137022317-38433f51-0e5e-405c-95e5-76afae5de7bb.png)

Comprobamos que, efectivamente, está publicada:

![image](https://user-images.githubusercontent.com/88755387/137023222-cb8c0616-a568-4cd1-a290-5257ee6ac963.png)

Una vez que tenemos todo esto listo vamos a crear un archivo llamado `index.html` el cual vamos a rellenar con cualquier cosa para comprobar que efectivamente hemos realizado un subdomain takeover.

```html
<h1> H4cked by h4ns, this is a test!! </h1>
```

Si nos damos cuenta, a la hora de crear el archivo html, vemos que hay un archivo llamado CNAME el cual contiene el nombre de nuestro subdominio al cual estamos apuntando.

Una vez que hemos hecho todo esto, ya nos podemos meter en dicha página.

![image](https://user-images.githubusercontent.com/88755387/137023496-de0c6f85-ddbd-457d-9821-28386963b90b.png)

Hemos logrado realizar un subdomain takeover :laughing:.

Esto no habría sido posible sin la ayuda de [Víctor García](https://twitter.com/takito1812) el cual estuvo enseñándome de que iba este ataque y como podía ser explotado.























