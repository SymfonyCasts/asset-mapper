# CSS y JS específicos de la página

Visita `/admin` ¡Sorpresa! Tenemos una sección de administración en nuestro sitio. Bueno... más o menos. Es sólo un gran rectángulo, pero representa a un administrador ficticio. ¿Por qué? Bueno, supongamos que tenemos algunos CSS y JS que sólo se necesitan aquí. Si escribimos eso de la forma normal y en los archivos normales, ese código se va a descargar en todas partes, incluso cuando alguien vaya al frontend de nuestro sitio. Eso, como mínimo, es un despilfarro. Una forma mejor es descargar el CSS y el JS del admin sólo cuando se visita el área del admin

Mi forma favorita de hacerlo es con los controladores lazy Stimulus, de los que ya hemos hablado. Pero otra opción es crear un conjunto extra de CSS y JavaScript que se carguen explícitamente sólo en estas páginas. Veamos cómo hacerlo con AssetMapper.

Si utilizáramos Webpack Encore, abriríamos el archivo `webpack.config.js` y añadiríamos una segunda entrada. Eso daría lugar a un nuevo archivo CSS y JavaScript. En AssetMapper, podemos hacer algo muy parecido.

## Crear un nuevo archivo CSS

Empecemos por el CSS, que es bastante sencillo. En el directorio `assets/styles/`, crea un archivo `admin.css` y, para ver si las cosas funcionan, añade `.admin-wrapper`con algo de relleno X-Y. 

[[[ code('bbe80d9cf4') ]]]

Eso añadirá un poco de espacio aquí. A continuación, entra en la plantilla de esta página - `templates/admin/dashboard.html.twig` - y, justo aquí, añade esa clase: `class="admin-wrapper"`.

[[[ code('b7696d4790') ]]]

En este punto, el nuevo archivo `admin.css` está técnicamente disponible públicamente... porque está en el directorio `assets/`. Pero, aún no lo estamos utilizando. Para ello, necesitamos una etiqueta de enlace.

Esto no tiene nada de especial. Di `{% block stylesheets %}` y `{% endblock
%}` para anular el bloque de la plantilla padre. Luego llama a `{{ parent() }}` para incluir lo normal y, aquí abajo, añade `<link rel="stylesheet"` apuntando a`asset('styles/admin.css')`. Y... déjame arreglar mi errata aquí arriba. Eso es lo que queremos.

[[[ code('79f2e96dbd') ]]]

De vuelta al sitio... ¡sí! El CSS se está aplicando: tenemos relleno extra. Refrescantemente sencillo.

## Crear un archivo JavaScript específico de la página

Pero... ¿qué pasa con JavaScript? Una vez más, empezaremos como en Encore. Crea un nuevo archivo... quizá junto a `app.js` llamado `admin.js`. Añade`console.log('admin.js file')` para que podamos ver si se está cargando.

[[[ code('4fb4e62029') ]]]

Al igual que con el archivo CSS, este archivo ya está disponible públicamente... pero nada lo está cargando. Recuerda: el archivo `app.js` se carga gracias a esta línea`<script type="module">` de aquí abajo que importa `app`. Lo obtenemos automáticamente, en `base.html.twig`, a través de la función Twig `importmap()`.

Entonces... ¿hay alguna forma de decirle a esta función que importe también nuestro archivo `admin.js`? En realidad, ¡no! ¿Por qué? Principalmente porque... ¡es muy fácil añadirlo nosotros mismos!

Observa: en `dashboard.html.twig`, pon `{% block javascripts %}`, `{% endblock%}`, y luego `{{ parent() }}`. Debajo, añade una etiqueta `<script>` con `type="module"`. Ahora vamos a codificar como si estuviéramos en un archivo JavaScript. Di `import` y luego la ruta al archivo JavaScript. Efectivamente, queremos algo como - `/assets/admin.js`. Pero, por supuesto, para obtener la ruta real utilizamos la función `asset()` y pasamos la ruta lógica: `admin.js`.

[[[ code('369c9b0192') ]]]

¡Ya está! ¡Vamos a probarlo! Actualiza y comprueba la consola. ¡Ya está! ¡Nuestro archivo `admin.js`se está cargando! Si echas un vistazo a la fuente de la página... aquí abajo... sí. Puedes ver `<script type="module">` de la función `importmap()` donde dice`import 'app'`. Y, después, importamos `admin.js` a través de su ruta.

El original es sólo `import 'app'`... porque confiamos en el`importmap` para mapearlo a su URL. Eso está bien... pero en realidad no es necesario. Poner la ruta aquí también funciona bien. Eso es lo que hacemos por simplicidad.

Una de las cosas que hemos visto en este capítulo es que todo lo que hay en el directorio `assets/` se expone públicamente... ¡que es para lo que sirve AssetMapper! Pero a veces puedes tener algunos archivos que quieras poner en ese directorio, pero mantenerlos privados. Veamos a continuación la función de exclusión de AssetMapper y otras opciones de configuración.
