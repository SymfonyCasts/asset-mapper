# Depurando

A veces las cosas no funcionan. Pero, con AssetMapper, hay algunos signos reveladores cuando las cosas van mal. Veamos algunas de las formas más comunes en que las cosas pueden... ponerse raras.

## ¿Error tipográfico en la ruta lógica? Falta una ruta versionada

Una de las formas más rápidas de estropear las cosas es en`/templates/vinyl/homepage.html.twig`: utilizar la función `asset()` y pasar una ruta no válida. Recuerda que en `assets/images/`, tenemos `penguin.png`. Por tanto,`images/penguin.png` es su "ruta lógica" en AssetMapper. Digamos `images/`, pero luego `duck.png`.

[[[ code('74046520d7') ]]]

Obviamente, ésta no es la ruta correcta... ni siquiera el animal correcto. Así que no es de extrañar que, en la página de inicio, obtengamos un 404. La clave de este 404, si nos fijamos en la consola, es su ruta de aspecto sospechoso. Fíjate bien: ¡no hay versión en el nombre del archivo! Esto nos dice que esta ruta no se encontró en ninguno de los directorios de AssetMapper: es una ruta lógica no válida. Por tanto, AssetMapper la ignoró y devolvió el nombre de archivo sin versionar.

Para ayudarte a visualizar las rutas lógicas válidas, recuerda que puedes ejecutar:

```terminal
php bin/console debug:asset
```

Aquí arriba, esto es todo lo que puedes pasar a la función `asset()`. Y aquí está `images/penguin.png`. Si en su lugar ponemos aquí `images/penguin.png`... ahora funciona.

[[[ code('a73371eb3b') ]]]

La clave que hay que buscar es el hash de la versión en el nombre del archivo. Si no está ahí, AssetMapper no ha podido encontrar tu ruta.

## Rutas de importación no válidas

Otro error común es desordenar una importación. Como... quizás en`styles/app.css`, escribimos mal una parte de esta imagen `url()`. O, en `app.js`, al importar `vinyl.js`, nos olvidamos del `.js` al final.

[[[ code('07d535518f') ]]]

¡Accidentes como éste nos dan el mismo resultado que el primer error! Cuando actualizamos, obtenemos un 404. Puedes verlo aquí. Pero de nuevo, la clave es que falta el hash de la versión. Eso es señal de que no se pudo encontrar la ruta, por lo que no pudo ser manejada por AssetMapper.

En este caso, la ruta no válida se encuentra en `app.js`. Cuando estamos dentro de una plantilla y utilizamos la función `asset()`, pasamos la ruta lógica a un archivo. Pero si estamos dentro de `app.js` o `app.css`, en lugar de la ruta lógica, utilizamos la ruta relativa. Esto es por diseño. Conseguimos codificar dentro de estos archivos como si AssetMapper no existiera. No necesitamos pensar en rutas lógicas, sólo pensamos:

> ¿Qué ruta relativa utilizaría si todos estos archivos se sirvieran directamente
> a mi navegador?

En cualquier caso, si falta el hash de la versión, tenemos una ruta no válida, que podría ser una ruta lógica no válida en una plantilla o una ruta relativa no válida en alguna importación.

## Ver una lista de rutas no válidas

Por cierto, hay una forma casi oculta de ver si aparece alguna importación no válida en algún lugar de tu código. Primero, Ejecuta:

```terminal
php bin/console cache:clear
```

Eso limpia la caché de Symfony, por supuesto, pero también limpia una caché interna en AssetMapper. Ahora, cuando ejecutemos

```terminal
php bin/console debug:asset
```

reconstruye internamente la caché de todos esos activos. Cuando lo hace, analiza nuestros archivos e informa de cualquier importación que falte. ¿Lo ves?

> ADVERTENCIA No se puede encontrar el activo `./lib/vinyl` importado de `assets/app.js`.

Y en este caso, incluso recibimos un mensaje adicional:

> Prueba a añadir ".js" al final de la importación

Buena idea. Si volvemos a añadir ese `.js`... las cosas vuelven a funcionar.

## Importar "paquetes" que faltan

La última forma habitual de estropear las cosas es utilizar una importación desnuda -una importación que no empiece por `./` o `../` - para algo que no aparece en tu `importmap`. Aquí, la intención es utilizar la biblioteca `bootstrap`... pero no la tenemos en nuestro `importmap`. El error exacto variará en función de tu navegador, pero en mi caso dice:

> Error al resolver el especificador de módulo "bootstrap". Las referencias relativas deben empezar por
> "/", "./" o "../".

Traducción:

> ¡Oye! Si estás intentando hacer referencia a una ruta relativa, has olvidado la parte `./` o `../`
> parte, ¡tonto! Pero si estás intentando importar un paquete, has olvidado
> ¡añadirlo a tu mapa de importación!

La solución suele ser ejecutar: `php bin/console importmap:require` para añadir ese paquete.

Lo siguiente: ¿qué pasa si tienes un montón de CSS o JavaScript que sólo quieres cargar en una única página o sección de tu sitio, como una sección de administración? ¿Cómo podemos organizar las cosas para no tener que cargar todo ese código en cada página?
