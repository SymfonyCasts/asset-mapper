# Mapear Activos

AssetMapper no es gran cosa. Seguro que se viste genial y tiene buenos movimientos de baile, pero en realidad es bastante sencillo. Tiene dos características principales.

Característica número uno: configuramos "rutas" -como el directorio `assets/` - y hace públicos los archivos que contiene.

Veámoslo en acción. Si te has descargado el código del curso, deberías tener un directorio`tutorial/` con un importante archivo `penguin.png` en su interior. Cópialo. Dentro de `assets/`, podemos organizar las cosas como queramos. Así que vamos a crear un directorio`images/` y a transportar allí a nuestro pingüino.

Ahora, recuerda, sin la magia de AssetMapper, los únicos archivos a los que nuestro navegador debería poder acceder son los que están dentro del directorio `public/`. Así que debería ser imposible añadir una etiqueta `img` que cargue nuestro pingüino. Pero... es posible.

## Utilizando la "ruta lógica

Dirígete a, qué tal, `templates/base.html.twig`. En cualquier lugar -iré por encima del bloque`body` - añade un `img` con `src="{{ asset() }}"` pasándole la ruta a nuestro archivo relativa al directorio `assets/`. Así que `images/penguin.png`.

[[[ code('262e466110') ]]]

Ya está. Esto se conoce como la "ruta lógica" al activo. Como hemos apuntado AssetMapper al directorio `assets/`, podemos referirnos a cosas dentro de él a través de su ruta relativa a esa raíz.

Y hay una forma estupenda de ver todos los activos que están en las rutas de AssetMapper yendo al terminal y ejecutando:

```terminal
php bin/console debug:asset
```

¡Genial! En primer lugar, en la parte superior, muestra las rutas de AssetMapper, incluido el directorio `assets/`. Este proyecto también tiene instalado Pagerfanta. Y ya estamos viendo cómo los bundles pueden añadir sus propias rutas AssetMapper para que sus propios archivos estén disponibles públicamente. Esto no será importante para nosotros, pero podríamos dirigir el navegador a cualquier archivo dentro de ese directorio del bundle.

A continuación, vemos nuestro archivo de imagen, nuestro archivo CSS y nuestro archivo JavaScript. Estas son sus rutas en el sistema de archivos y estas son sus rutas lógicas.

## Nombres de archivo versionados

La cuestión es que, utilizando la función `asset()` y la ruta lógica a un activo, cuando actualizamos... ¡funciona! ¡Woh! Y si inspeccionamos el elemento, ¡mira la URL! ¡Contiene un hash de versión en el medio! En realidad voy a ver la fuente de la página... es un poco más fácil de ver.

Así que no sólo `penguin.png` está disponible públicamente, sino que la ruta no es sólo`penguin.png`: contiene un hash de versión. Si modificáramos el archivo fuente `penguin.png`-por ejemplo, poniéndole una pajarita chula- el hash de la versión cambiaría automáticamente, obligando a cualquiera que utilizara nuestro sitio a descargar el archivo nuevo. ¡Booya!

Así es como se carga `app.css` En la parte superior, la etiqueta de enlace utiliza`asset('styles/app.css')`, que es la ruta lógica en AssetMapper a ese archivo. Y así también sale con un bonito nombre de archivo versionado.

## ¿Cómo se hacen públicos los archivos?

Vale, pero ¿cómo funciona esto? Si eres como yo, querrás saber cómo se hacen las salchichas. Bueno, en el entorno `dev`, funciona gracias a un oyente de eventos del núcleo... básicamente un elegante controlador interno de Symfony.

Por ejemplo, cuando el navegador carga esta imagen, esa petición pasa por Symfony. Ve que estamos intentando cargar `/assets/images/penguin-versionhash.png`, encuentra el archivo fuente y lo sirve.

¡Podemos probarlo! En la pestaña principal, haz clic en cualquier icono de la barra de herramientas de depuración web para entrar en el perfilador y, a continuación, haz clic en "Últimas 10" para ver las 10 peticiones más recientes a través de Symfony. Y ahí está: la petición que sirvió la imagen del pingüino. Adorable.

En producción, cargar nuestros archivos a través de Symfony no sería lo suficientemente rápido. Así que, en su lugar, durante el despliegue, ejecutarás un nuevo comando de consola:

```terminal
php bin/console asset-map:compile
```

Hablaremos más sobre el despliegue más adelante. ¡Pero esto es realmente genial! Copia cada archivo de cada ruta AssetMapper en el directorio `public/assets/` utilizando su nombre de archivo versionado. Y ya está.

De repente, este archivo ya no está siendo servido por Symfony: ¡estamos viendo un archivo real, físico! En `public/assets/`, ¡sí! Podemos ver los archivos finales en todo su esplendor.

Pero... mientras desarrollamos, elimina ese directorio para que todo siga cargándose dinámicamente.

## Trasladar los favicons a AssetMapper

Y ya que estamos aquí, ¿ves esos favicons dentro del directorio `public/`? Estamos enlazando con ellos en la parte superior de `base.html.twig`. Eso funciona perfectamente: la función`asset()` puede seguir haciendo referencia a cosas dentro del directorio `public/`.

[[[ code('d187eec649') ]]]

Pero... ¡sin apenas trabajo, podemos añadirles el versionado libre de activos! Paso 1: muévelos al directorio `assets/images/`. Paso 2: añade a cada ruta el prefijo`images/` para obtener su ruta lógica.

Y... así de fácil, seguimos viendo el favicon aquí arriba... pero lo que es más importante, si vemos el código fuente de la página, ¡ahora están versionados!

A continuación, profundicemos un poco más en los archivos CSS. Por ejemplo, ¿cómo podemos referirnos a imágenes de fondo desde dentro de CSS... si el nombre final del archivo está versionado?
