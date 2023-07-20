# Importar archivos específicos de un paquete

A veces, en lugar de importar un paquete en sí, puedes querer importar sólo una parte de él: como un archivo concreto. Lodash es un buen ejemplo de ello.

## Sin agitar el árbol en el navegador

Pero antes de llegar ahí, en lugar de importar todo desde `lodash`, deberías poder decir `import { camelCase } from 'lodash'`. Entonces, aquí abajo, utilizarías directamente `camelCase`.

[[[ code('910a119d46') ]]]

Sin embargo, cuando nos desplazamos e intentamos esto... ¡error!

> El módulo solicitado `lodash` no proporciona una exportación llamada `camelCase`.

Esto debería funcionar... y la razón por la que no lo hace es complicada. Básicamente, debido a la forma en que esta biblioteca específica empaqueta su módulo, no puedes importar funciones específicas como ésta. Sin embargo, funcionará con la mayoría de los demás paquetes.

Por ejemplo, si dices `import { Modal } from 'bootstrap'` (si utilizas Bootstrap), funcionará. Bootstrap empaqueta sus archivos correctamente.

Sin embargo, utilizar esta sintaxis no siempre es lo ideal con AssetMapper.

El problema es el siguiente. Si pasáramos este código por Encore, Encore haría algo llamado "sacudir el árbol". Vería que sólo estamos importando`camelCase` desde `lodash`. Y así, en el JavaScript final, sólo nos daría el código de `camelCase`, no todo el paquete `lodash`.

En un entorno de navegador, si importas `import` de `lodash`, obtendrás todo `lodash`... aunque sólo estés importando una parte de él. Ahora bien, puede que no sea para tanto. La compilación completa de `lodash` sigue siendo de sólo 24 kilobytes. Pero, ¿y si estamos utilizando un paquete grande... pero sólo necesitamos importar una cosa concreta?

## Importar un archivo concreto

Muchas veces, hay un archivo específico que podemos importar, como `/camelCase`. Normalmente encontrarás detalles sobre estos archivos en la documentación... aunque también puedes ir a buscarlos. Vuelve a JSDelivr... y aquí abajo, busca "lodash". A continuación, haz clic en "Archivos" para ver todos los archivos que forman parte de este paquete.

Para `lodash`, es una lista enorme... porque se trata de una biblioteca enorme. Uno de ellos es `camelCase.js`.

¡Vale! Así que vamos a intentar importar `lodash/camelCase`.

[[[ code('0b0f95b8ba') ]]]

No voy a incluir el `.js`... pero de todas formas no va a funcionar. Observa: cuando actualizamos... ¡error!

> Error al resolver el especificador de módulo `lodash/camelCase`. Las referencias relativas deben
> empezar por "/", "./" o "../"

Este error significa que estamos importando algo mediante una importación "desnuda", y no se encontró en el `importmap`. Si "Vemos la fuente de la página", sí tenemos un `importmap`para `lodash`, pero no `lodash/camelCase`. Sí, esa coincidencia se hace exactamente. Vale, hay una forma de hacer una coincidencia, más o menos, "difusa" - `lodash/*` - pero yo no la uso.

La cuestión es: si quieres utilizar `lodash/camelCase`, debes añadirlo a tu mapa de importación, no `lodash`.

Ejecuta: busca tu terminal y ejecuta:

```terminal
php bin/console importmap:remove lodash
```

Eso eliminará `lodash` de `importmap.php` y borrará el archivo de`assets/vendor/`. 

[[[ code('24d74b7889') ]]]

¡Qué bien! Ahora ejecuta `./bin/console importmap:require` con el nombre del paquete / la ruta que quieras: `lodash/camelCase.js`.

```terminal-silent
php bin/console importmap:require lodash/camelCase.js
```

`camelCase.js` es el nombre del archivo en la CDN. Pero te darás cuenta de que, muchas veces en los documentos, se hace referencia a `lodash/camelCase` sin el `.js`. Y en este caso, puedes dejar el `.js`: tú decides. Eso funciona porque jsDelivr es amigable y hace que funcionen ambas versiones de la URL.

¿El resultado del comando? El mismo que antes Obtenemos una nueva entrada en`importmap.php` que coincide con lo que queremos importar y establecer en una URL. 

[[[ code('39bbe655b2') ]]]

Copia esa URL para que podamos verla. ¡Ya está! Es el código de sólo `camelCase.js`.

Y cuando probamos la página... ¡funciona!

Estas son las conclusiones: si necesitas importar un archivo concreto de un paquete, puedes hacerlo: sólo tienes que pasar el nombre del paquete + la ruta del archivo a `importmap:require`.

A continuación, ¡añadamos Stimulus a nuestra aplicación!
