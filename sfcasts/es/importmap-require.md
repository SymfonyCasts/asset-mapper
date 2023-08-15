# importmap:require - Libs JS de terceros

En nuestro código, podemos utilizar declaraciones `import` con rutas relativas, clases ES6: todo a lo que estamos acostumbrados. Es lo de siempre. Excepto, ¿cómo podemos utilizar paquetes de terceros?

Como vimos antes, podemos importar cosas a través de una URL completa, como `import _ from`, y yo pegaré la URL CDN que usamos antes. Hecho esto, el resto es normal: añade `_.camelCase()` al registro.

[[[ code('74a43597cd') ]]]

Si refrescamos y comprobamos la consola... funciona. Pero no me gusta! No quiero tener que incluir esta URL loca en todas partes donde utilice `lodash`. ¿Y qué pasa si actualizamos Lodash... y tengo que cambiar la URL en 10 archivos diferentes? ¡Qué pena!

## importmap:require para obtener paquetes de nodos

Si utilizáramos un sistema de compilación, como Webpack, podríamos simplemente decir

```terminal
yarn add lodash
```

o

```terminal
npm install lodash
```

No estamos utilizando `yarn` ni `npm`, pero podemos hacer casi lo mismo. En el terminal, abre una nueva pestaña y ejecuta`php bin/console importmap:require` seguido del nombre del paquete NPM que queremos: `lodash`:

```terminal-silent
php bin/console importmap:require lodash
```

¡Listo! Ha añadido `lodash` a `importmap.php` y nos dice que podemos utilizar el paquete como de costumbre. Esto significa que podemos decir `import _ from 'lodash'`... y todo funcionará correctamente.

[[[ code('eae2dcf77e') ]]]

¿Cómo? Cuando ejecutamos el comando, hizo un pequeño cambio: añadió esta sección a `importmap.php`. Y aunque esto sea genial, no es magia. Entre bastidores, el comando fue a la CDN de JSDelivr, encontró la última versión de `lodash`, y luego añadió el conjunto de claves `lodash` a esa URL.

Si te acercas y miras la fuente de la página... ¡no hay sorpresa! ¡Tenemos una nueva entrada `lodash`dentro de la `importmap`! Cuando nuestro navegador ve `import _ from 'lodash'`, busca `lodash` dentro de `importmap`, encuentra esta URL y la descarga desde allí. ¡Nuestro navegador es el héroe!

## Informar a tu editor sobre los paquetes

Una pega es que no tenemos autocompletado en nuestro editor. Dice "Módulo no instalado". Y si digo `_.`... en realidad no funciona. Autocompleta `camelCase`... pero sólo porque lo estoy usando aquí abajo.

Espero que PhpStorm lo soporte mejor pronto. Hay una solución, pero es un poco manual. Copia el paquete, entra en `base.html.twig` y añade una etiqueta temporal`<script>` que apunte a esto. Pulsa "alt" + "enter" y selecciona "Descargar biblioteca". Esto lo descarga en la sección "Bibliotecas externas" de aquí abajo: `/lodash`.

Vale, elimina la etiqueta `script`. De nuevo en `app.js`, seguirá subrayando la importación como si no supiera lo que es, pero autocompleta cuando usamos `_.` algo. Por ejemplo, `tail()` es de `lodash`.

## Actualizar paquetes

¿Qué pasa con la actualización de las versiones de los paquetes dentro de `importmap`? Vaya, ¡hay un comando para eso!

```terminal
php bin/console importmap:update
```

Eso hará un bucle a través de cada paquete y actualizará su URL a la última versión. Ésta ya es la última versión... pero si la cambiamos a `.19`... y luego ejecutamos el comando `update`... retrocede hasta `.21`. El comando podría ser más flexible -como permitirte actualizar sólo un paquete, o tener algunas restricciones de versión- y esas cosas podrían añadirse en el futuro.

[[[ code('119893197f') ]]]

## Descargar paquetes localmente

Por último, si no quieres depender de la CDN, no tienes por qué hacerlo. Para evitarlo, cuando necesites el paquete -o en cualquier momento posterior- pasa la opción `--download`:

```terminal-silent
php bin/console importmap:require lodash --download
```

En `importmap.php`, esto sigue mostrando la URL de origen a la CDN, pero descarga ese archivo en un directorio `assets/vendor/`. Este `downloaded_to` apunta a la ruta lógica de ese archivo.

[[[ code('157bb3da90') ]]]

¿Cuál es el resultado? Cuando vamos y actualizamos .... y "Vemos la fuente de la página"... ¡el `importmap`apunta ahora al archivo local! Ya no dependemos de la CDN.

Pero... ¿y ahora qué? ¿Confirmamos este archivo `vendor/lodash.js`? La respuesta es... sí. Al menos en este momento, esa es la única forma de versionar ese archivo y mantenerlo en tu repositorio.

Así que, incluso sin npm ni yarn, podemos utilizar cualquier paquete npm que queramos. Pero a veces, en lugar de importar un paquete entero, puede que sólo queramos importar un archivo concreto. Hablemos de cómo podemos hacerlo a continuación.
