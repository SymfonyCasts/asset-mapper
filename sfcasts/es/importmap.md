# JavaScript y mapa de importación

Elimina la etiqueta `<img>`, para que podamos ver nuestra página normal. No te preocupes por nuestro pequeño pingüino: seguimos teniéndolo aquí arriba, en el logotipo.

Cuando actualicemos la página, observa que tenemos un mensaje `console.log()`... que dice que procede de `assets/app.js`. Si nos dirigimos a `assets/app.js`... ¡sí! ¡Ahí está!

[[[ code('7e156a5fbe') ]]]

## Cómo se carga assets/app.js

Sabemos que aquí podemos escribir código ES6 moderno, así como importar otros archivos. Vamos a hacer todo eso. Pero primero: ¿Cómo y por qué se ejecuta este archivo? Nuestro CSS se está cargando gracias a esta bonita y aburrida etiqueta`<link>`. No vemos una etiqueta `<script>` para `app.js`... pero sí vemos esta función`importmap()`. Y ahí está la clave.

De vuelta al sitio, mira la fuente de la página. Aquí abajo... esto es lo que añade `importmap`. Vamos a hablar de cada parte, pero lo más importante ahora mismo está en la parte inferior:

```js
<script type="module">import 'app';</script>
```

Antes, cuando creamos un archivo `app.js` dentro del directorio `public/`, éste es casi exactamente el código que escribimos para cargarlo. Utilizamos `import` y luego la ruta a ese archivo. Pero... esta vez, sólo dice `app`. ¿No debería decir algo como`/assets/app.12345.js`"? ¿Cómo sabe que `app` se refiere a la versión final de este archivo? Aquí es donde brilla la parte `importmap`, aquí arriba.

## El maravilloso mapa de importación

Esta sección se genera a partir de un archivo `importmap.php` dentro de nuestro proyecto. El archivo no es superinteresante todavía: será más útil pronto, cuando hablemos de JavaScript de terceros. Pero tiene esta clave `app` que apunta a nuestro archivo `assets/app.js` utilizando su ruta lógica.

[[[ code('1d5eea403c') ]]]

Gracias a eso, este `<script type="importmap">` se vuelca en la página. Cuando importas algo que no empieza por ".", "/" o "../", se denomina importación desnuda. Normalmente lo vemos en bibliotecas de terceros. En el entorno del navegador, cuando ve una "importación desnuda", tu navegador busca un `importmap` en la página para encontrar una entrada que coincida. Nuestro navegador ve `import 'app'`, encuentra esta clave aquí, y esa es la ruta que descarga. De hecho, copia esta ruta de aquí y la pega aquí. Por eso se está ejecutando nuestro archivo `app.js`: ¡es un trabajo en equipo entre el `importmap` y el`<script type="module">` extra que arranca nuestra aplicación!

Lo mejor de `importmap` es que no es una cosa de Symfony: es una cosa de Internet. Es cómo funciona tu navegador. Tenemos este archivo `importmap.php`, que es una cosa de Symfony. Pero una vez que está en la página, tu navegador es la estrella.

## El shim importmap + Navegadores antiguos

Y `importmap` funciona en... la mayoría de los navegadores. Si vas a "caniuse.com" y buscas "importmap"... actualmente funciona en cerca del 81% de los navegadores. Eso sería un gran problema, si no fuera porque la función `importmap()` también vuelca un calco. Puedes verlo aquí. Gracias a esto, si un navegador no soporta `importmap`, esto añade esa funcionalidad. Así que funcionará.

## Importar archivos JavaScript relativos

Entra en `app.js`: vamos a escribir algo de código moderno. En `assets/`, primero crea un nuevo directorio llamado `lib/`. Y dentro de él, un nuevo archivo llamado `vinyl.js`. Puedes organizar las cosas como quieras, y éste es un ejemplo de cómo aislar algo de código en su propio archivo.

Voy a pegar la misma clase que teníamos antes. De vuelta a `app.js`, importa esto: `import Vinyl` y puedo pulsar "tab" para autocompletar la parte`from './lib/vinyl'`. Instálalo utilizando el mismo código que antes... y luego `console.log(mix.describe())`.

[[[ code('3c4a6ec1d5') ]]]

[[[ code('a84807bda2') ]]]

## Utilizar .js al importar

¡Me encanta! Estamos codificando como siempre y utilizando `./` para importar. Pero cuando pasamos y actualizamos... no funciona. Mira el 404 `/assets/lib/vinyl`
procedente de `app.js`.

Entonces... ¿qué está pasando aquí? Hablaremos más adelante sobre depuración, pero aquí tienes una pista: si alguna vez notas que tu navegador intenta descargar una ruta que no incluye la parte "versión" en el nombre del archivo, algo va mal en tu ruta... y deberías comprobar si hay errores tipográficos.

Nuestro problema es que necesitamos añadir el `.js`. Resulta que dejar el `.js`desactivado es una cosa de Node... y funciona si estás programando en Node. Pero en verdaderos entornos JavaScript, como en tu navegador, sí necesitas incluirlo.

[[[ code('6ea8840f69') ]]]

Si refrescamos ahora... ¡ya está! En realidad fue culpa de mi editor que faltara el `.js`cuando lo autocompletó. Afortunadamente, ¡podemos arreglarlo! Entra en la configuración de tu PhpStorm y busca "usar extensión de archivo". En "Estilo de código" y "JavaScript", cambia "Usar extensión de archivo" a "Siempre".

Esta vez... si decimos `import Vinyl` y pulsamos "tabulador", ¡bien! Obtenemos el `.js`.

## Entradas automáticas de Importmap

Pero la diversión no termina: hay algo interesante que ocurre entre bastidores. Haz clic en este `console.log()`... como una forma fácil de ver el origen del archivo final `app.js`.

Sí, su contenido es exactamente igual al del archivo original, incluido el`import from './lib/vinyl.js'`. Sólo hay un problema: ¡ese no es el nombre final del archivo `vinyl.js`!

Ve a las herramientas de red, selecciona "JS" y busca "vinyl". Todos los archivos servidos por AssetMapper tienen una parte versionada en su nombre de archivo, y lo vemos para `vinyl.js`. Pero entonces... ¿cómo demonios lee nuestro navegador `./lib/vinyl.js` y sabe que debe descargar este nombre de archivo tan largo?

La respuesta, si ves la fuente de la página, es... redoble dramático de tambores... el `importmap`. Y esto me encanta. El `importmap` se construye a partir de dos fuentes. La primera fuente es obvia: `importmap.php`. Y pronto añadiremos más entradas. La segunda fuente es más sutil. Cada vez que nuestro JavaScript importa otro archivo JavaScript utilizando una ruta relativa, ese archivo importado se añade automáticamente.

Esto es poderoso. Significa que nuestro código final puede tener el aspecto original:`./lib/vinyl.js`. Pero gracias a `importmap`, nuestro navegador descargará inteligentemente el archivo real con la parte de la versión larga en el nombre. Esto es realmente un detalle interno, pero está bien ver cómo funciona.

Vale, hemos hablado un poco de `importmaps`... pero no hemos visto su mayor superpotencia: utilizar paquetes de terceros. Exploremos eso a continuación.