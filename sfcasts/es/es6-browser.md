# Hacer JS moderno directamente en tu navegador

Antes de hablar de cualquier cosa relacionada con Symfony, vamos a reducir las cosas al mínimo y demostrar que podemos codificar JavaScript moderno, directamente en nuestro navegador.

## Carga directa de JavaScript

Ve directamente al directorio `public/` y crea un nuevo archivo `app.js`. Para empezar, escribe un mensaje en `console.log()`.

Esto no será procesado por Symfony ni nada parecido. En `templates/base.html.twig`, aquí arriba en el bloque `javascripts`, aunque eso no supone ninguna diferencia, añade una aburrida etiqueta `<script>` para esto: `<script src="{{ asset('app.js') }}">`. Estoy utilizando la función `asset()`... pero eso tampoco hace nada.

[[[ code('36691e46e5') ]]]

Vale, ve al navegador, abre tu Consola y... actualiza. Ahí está el log! Es digno de snooze, pero funciona.

## Escribir JavaScript moderno

¡Es hora de hacer las cosas interesantes! De vuelta en `app.js`, copia el nombre de la mezcla. Vamos a crear una clase `class MixedVinyl` con un constructor y algunas propiedades. Esto utiliza la sintaxis de clase introducida en ES6, o ECMAScript 6... básicamente la versión "6" de JavaScript. Oirás hablar mucho de ES6 porque la mayoría de las funciones modernas a las que estás acostumbrado provienen de esta versión, lanzada allá por 2015.

[[[ code('0a5e0272ff') ]]]

En el método `describe()`, aprovecho la interpolación de cadenas -otra función moderna de ES6- para devolver la cadena. A continuación, utiliza esto `const` - otra función de ES6 - `mix = new MixedVinyl()` y pasa el nombre de la mezcla y el año. Por último,`console.log(mix.describe())`.

[[[ code('1cf2eec3fa') ]]]

¡Genial! Este es el tipo de código que me gusta escribir cada día. Por desgracia, ¡también es el tipo de código que históricamente se les ha atragantado a los navegadores!

Así que, normalmente, tendríamos un sistema de construcción como Encore que leería este código moderno y lo reescribiría en JavaScript antiguo... para que funcionara en nuestro navegador. Pero... ¡tachán! ¡Ya funciona en nuestro navegador! No necesitamos hacer nada. Y no es sólo porque esté utilizando un navegador nuevo. Esto va a funcionar en todos los navegadores.

Si alguna vez tienes dudas, ve a https://caniuse.com para comprobarlo. Busquemos "clase ES6". Sí, básicamente es compatible con todo... excepto con IE 11, que está muerto.

## Uso de "import" en el navegador

Pero, ¿qué pasa con la declaración `import`? Copia `class MixedVinyl` y luego crea otro archivo directamente dentro de `public/` llamado `vinyl.js`. Pega esto y luego `export`: `export default class`.

[[[ code('92a5ef4225') ]]]

Vuelve a `app.js`, `import MixedVinyl from` y, al igual que hacemos en Encore, utiliza la ruta relativa: `./vinyl.js`.

[[[ code('da3b7f12da') ]]]

Sin embargo, fíjate en que estoy incluyendo la extensión de archivo `.js`... lo cual puedes hacer en Encore, pero no es necesario. Más adelante hablaremos de ello, pero ha sido a propósito.

## Importar como módulo

Entonces... ¿mi navegador admite la declaración `import`? Averigüémoslo Actualiza:

> No se puede utilizar la sentencia import fuera de un módulo

Vale, no es un "código rojo", es más bien un "código naranja". Vuelve a`base.html.twig`. Cuando oigas la palabra "módulo", se refiere a archivos que aprovechan `export` y `import`. Y si quieres que tu JavaScript pueda utilizarlos, tienes que cargar el archivo original "como módulo". Es un cambio sencillo. Copia la función `asset()` y pon ahora `<script type="module">`. Entonces, en lugar de `src`, dentro, vamos a escribir algo de JavaScript en `import` nuestro archivo `app.js`.

[[[ code('25822ef79c') ]]]

Esto puede parecer una locura al principio, pero... simplemente estamos importando la ruta a nuestro archivo`app.js`. Al hacer esto, `app.js` se ejecutará exactamente igual que antes... pero como un "módulo"... lo que sólo significa que las declaraciones `import` y `export` "deberían" funcionar.

¿Funcionan? ¡Funcionan! OMG, ¡nuestro navegador admite la sentencia `import`! 

## Importar URLs de paquetes de terceros

Incluso podemos importar paquetes de terceros. Para encontrar uno, voy a utilizar mi CDN favorito: "jsDelivr". Lo utilizaremos bastante a lo largo del tutorial. Pero no es necesario que utilices la CDN de jsDelivr en tu código final. Es una réplica de cada paquete NPM... y por eso es un lugar conveniente para encontrar lo que necesitamos.

Busca el popular paquete "lodash". Cuando lo seleccionamos, nos muestra una etiqueta `<script>` que podríamos utilizar. Haz clic en "ESM", que es la abreviatura de módulos ECMAScript. Cuando codifiques con importaciones y exportaciones, querrás la versión ESM de un paquete: es una versión que "exporta" módulos correctamente.

Fíjate en la etiqueta `script`:

```js
<script type="module">
import lodash from '[...]'
</script>
```

¡Se parece mucho al código que tenemos aquí! No lo utilizaremos exactamente, pero voy a copiar la URL. Ahora vuelve a `app.js`. Para utilizar `lodash` podemos decir `import _ from` y pegar esa URL completa.

[[[ code('f3c52fc010') ]]]

Sí, importar desde una URL completa está totalmente permitido. O podríamos descargar este archivo localmente: Hablaré más sobre esto más adelante. A continuación, digamos `_.camelCase()` para llamar a uno de sus métodos.

¡Vamos a probarlo! Gira, actualiza y... ¡mira eso!. Aquí no hay sistema de compilación: sólo estamos jugando con archivos dentro del directorio `public/`. Y, sin embargo, estamos escribiendo JavaScript moderno, importando y exportando módulos y utilizando un paquete NPM de terceros. Eso es bastante asombroso.

## ¿Qué funciones faltan?

Sin embargo, quedan dos problemas. En primer lugar, importar paquetes utilizando la URL completa es molesto. Quiero poder decir `import from 'lodash'`. El segundo problema es el versionado de activos. Para tener un sistema eficaz, necesitamos que los archivos finales descargados por el navegador tengan hashes de versión en sus nombres de archivo, como`app.1234abcd.js`. Lo necesitamos para poder indicar a los navegadores que realicen un almacenamiento en caché a largo plazo. Y no podemos conseguirlo creando y sirviendo archivos directamente desde `public/`.

Estas son precisamente las dos cosas que el nuevo componente AssetMapper de Symfony nos ayudará a resolver. Pero quería empezar con JavaScript en bruto para que pudiéramos ver cómo... la mayor parte de lo que estamos haciendo no lo resuelve Symfony ni AssetMapper ni la IA: lo resuelve tu navegador y la web moderna.

Vale, vamos a eliminar estos dos archivos para no confundirme... y también vamos a eliminar el`import` dentro de `base.html.twig`. ¡No te preocupes! Pronto veremos todo ese código de otra forma.

A continuación: Instalemos AssetMapper y pongámoslo en marcha.
