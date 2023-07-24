# Añadir Stimulus

Podemos escribir JavaScript moderno en este archivo, podemos importar paquetes de terceros: somos libres de idear el código que queramos. Pero, si eres como yo, probablemente quieras utilizar Stimulus. Así que vamos a instalarlo.

Stimulus no es más que una biblioteca de JavaScript, así que podríamos decir

```terminal
php bin/console importmap:require '@hotwired/stimulus'
```

Luego sigue su documentación sobre cómo configurar las cosas.

## Instalar StimulusBundle

Pero Symfony tiene una integración especial con Stimulus. Así que en su lugar, ejecuta:

```terminal
composer require symfony/stimulus-bundle
```

StimulusBundle es un paquete relativamente nuevo que contiene algunos atajos de Twig que utilizaremos, como `stimulus_controller()`. Pero, lo que es más delicioso, tiene una receta que configurará nuestra aplicación para cargar controladores Stimulus sin esfuerzo.

Compruébalo: gracias a la receta, ahora tenemos un directorio `assets/controllers/`con `hello_controller.js` dentro. 

[[[ code('3f98852833') ]]]

Sin tocar nada más, abre `templates/vinyl/homepage.html.twig` y, justo después de `<h1>`, añade un nuevo `<div>`. Vamos a adjuntar el nuevo controlador `hello` a este elemento. Hazlo con `stimulus_controller()` - que es una de las nuevas funciones que vienen de StimulusBundle - pasando `hello`.

[[[ code('2f00c2c241') ]]]

Eso no puede funcionar ya... ¿verdad? Actualiza. Y funciona. Y abajo, en la consola, vemos registros sobre la inicialización de Stimulus y la conexión de nuestro controlador`hello`. Con sólo una línea `composer require`, ¡Stimulus está vivo!

## Cómo se carga Stimulus

Pongámonos nuestros sombreros de detective y profundicemos un poco más en cómo funciona esto y qué hizo realmente la receta. En `templates/base.html.twig`, éste es probablemente el cambio menos importante: añadió `ux_controller_link_tags()`. Hablaremos de ello en el próximo capítulo, cuando exploremos los paquetes UX. Pero, en resumen, si un paquete UX viene con su propio CSS, esto lo produce. Ahora mismo, no hace nada.

[[[ code('66246447cd') ]]]

Y lo que es más importante, la receta añadió un nuevo archivo `assets/bootstrap.js`. Y, en`assets/app.js`, espolvoreó algo de código para importar ese archivo. Así, se carga `app.js`, que importa `bootstrap.js`, y luego que importa `@symfony/stimulus-bundle`.

[[[ code('2b34113a1b') ]]]

Ooh, ¡eso es una importación desnuda! ¡No empieza por "../" ni "./"! Eso significa que nuestro navegador lo buscará en `importmap` para averiguar qué archivo debe cargar.

¡De acuerdo! Ve a abrir `importmap.php`. ¡Sorpresa! La receta ha añadido dos entradas nuevas: Una para la propia biblioteca `@hotwired/stimulus` y otra para `@symfony/stimulus-bundle`, que apunta a este `path` de aspecto extraño.

[[[ code('068db5a3ec') ]]]

Aquí arriba, cuando se utilice una CDN, la entrada tendrá una clave `url`. Cuando apunte a un archivo local, la entrada tendrá una clave `path`, que será la ruta lógica a un archivo en AssetMapper.

Pero, ¿a qué apunta esta extraña ruta? Gira hasta tu terminal y ejecuta:

```terminal
php bin/console debug:asset
```

Si coges un ascensor hasta arriba... ¡voilà! Cuando instalamos StimulusBundle, añadió una nueva "ruta de activos" a nuestro sistema, que apunta a`vendor/symfony/stimulus-bundle/assets/dist` y tiene un "prefijo de espacio de nombres". Esto significa que, para apuntar a un archivo de este directorio, la ruta lógica empezará por `@symfony/stimulus-bundle`.

Así que aquí, cuando decimos `@symfony/stimulus-bundle/loader.js`, nos estamos refiriendo a este archivo de aquí: `vendor/symfony/stimulus-bundle/assets/dist/loader.js`. Es una forma larga de decir que cuando importamos `@symfony/stimulus-bundle`, en realidad estamos importando este archivo `vendor/symfony/stimulus-bundle/assets/dist/loader.js`. El bundle expone ese archivo añadiendo la "ruta de activos" de AssetMapper, que permite a la receta añadir una entrada a `importmap.php` que apunte a él.

## Cómo se registran nuestros controladores

Bien, estamos cargando este archivo `loader.js`, y podemos verlo aquí. En tu navegador, actualiza... ve a tus herramientas de Red, y busca "cargador". Ahí lo tienes Ábrelo en una pestaña nueva.

Este código tiene funciones para iniciar la aplicación Stimulus y registrar los controladores de nuestra aplicación, como `hello_controller.js`. Pero... espera. Esto es sólo un archivo codificado. ¿Cómo es capaz de encontrar y cargar dinámicamente los archivos que viven dentro de nuestro directorio `assets/controllers/`?

La clave está arriba: `import`, `isApplicationDebug`, `eagerControllers`,`lazyControllers` de `./controllers.js`. Esto... es un poco de magia. Vuelve a las herramientas de Red y busca "controladores"... ahí está - `controllers.js`. Abre esta nueva pestaña. ¡Woh! Tiene `import controller_0` de`../../controllers/hello_controller.js`, que luego exporta a una variable llamada`eagerControllers`.

Este archivo es creado dinámicamente por el bundle. Si miramos en el directorio `vendor/`, `loader.js` es un bonito archivo estático. Pero si nos fijamos en `controllers.js`, ¡no se parece en nada a lo que tenemos en el navegador! Cuando se sirve este archivo, AssetMapper lo intercepta, mira dentro de nuestro directorio `assets/controllers/`, encuentra allí todos los controladores y devuelve contenidos dinámicos basados en ellos.

Observa. Crea otro archivo llamado `goodbye-controller.js` (puedes utilizar guiones o guiones bajos). Cambia el texto a `Goodbye controller!`.

[[[ code('3008b7ae80') ]]]

Es de esperar que, cuando actualicemos el archivo, veamos aparecer aquí el nuevo controlador. Y casi tienes razón. Lo que realmente ocurre es... ¡nada! Ningún cambio! O incluso podrías obtener un error 404. Eso es porque el contenido de este archivo acaba de cambiar y, por tanto, el hash también cambiará. ¡Estamos ante una versión desactualizada del archivo!

De vuelta al sitio, si actualizamos, deberíamos ver un nuevo archivo con un nuevo hash. No lo vemos... debido a un error de almacenamiento en caché que ya se ha solucionado. Para solucionarlo, voy a ejecutar:

```terminal
php bin/console cache:clear
```

Y luego actualizar. ¡Ahora veo que tiene un nombre de archivo diferente, y que el contenido ha cambiado dinámicamente para incluir `goodbye-controller.js`!

Así que ahí lo tienes, el emocionante viaje al corazón de cómo Stimulus y AssetMapper se hicieron mejores amigos. `bootstrap.js` carga un archivo que inicia Stimulus... y que carga automáticamente todo lo que hay dentro del directorio `assets/controllers/`... así como cualquier paquete UX de terceros en `assets/controllers.json`. Hablemos ahora de esos paquetes de terceros.
