# Almacenamiento en caché a largo plazo, compresión y combinación de archivos

¡Es hora de celebrarlo! Estamos en producción, y en realidad ha sido tan sencillo como asegurarnos de que Tailwind se estaba construyendo y ejecutar el comando `asset-map:compile`.

Ahora que estamos aquí, hay algunas cosas que debemos comprobar para asegurarnos de que nuestro sitio es rápido.

## 1) HTTP/2: Definitivamente lo necesitas

Lo primero es comprobar que tu servidor web utiliza HTTP/2. Ya hemos hablado de ello antes, así que esperemos que ya lo tengas en marcha.

## 2) Combinar archivos

Lo segundo es... bueno... en realidad no es nada. Sólo quiero señalar que no hay nada en AssetMapper que combine nuestros archivos para reducir el número de peticiones HTTP. Hablaremos más de esto en unos minutos, pero gracias a HTTP/2, casi seguro que no necesitas combinar tus archivos. Así que si estabas pensando que esto faltaba, ¡no es así! Es por diseño. ¡Qué bien! Una cosa menos.

## 3) Compresión / Minificación de archivos

¿Pero qué pasa con la minificación de nuestros archivos? Es cierto: ahora mismo, nuestros archivos se sirven sin minificar, y eso es un problema. Queremos que nuestros archivos CSS y JavaScript estén minificados... o al menos comprimidos. Y esta es la tercera cosa en la que debemos pensar.

Pero... esto es algo que puede hacer nuestro servidor web. Sí, si lo pides amablemente, deberías poder convencer a tu servidor web de que comprima tus archivos para que sean más pequeños cuando se envíen a través de la red. Esto es algo que admiten todos los servidores web, y lo hace automáticamente Platform.sh.

Así es como podemos verlo Ve a las herramientas de "Red" y selecciona JavaScript. Selecciona uno de los archivos y luego ve a "Encabezados". Haré esto un poco más grande. Vale, ¿ves esta cabecera de respuesta "Content-Encoding"? Eso es compresión. Este "br" significa algo llamado Brotli, que es más delicioso de lo que parece. Brotli es un formato de compresión avanzado. El otro valor común es `gzip`. Así que todos nuestros archivos estáticos ya están comprimidos Lo conseguimos gratis con Platform.sh, así que podemos tachar ese elemento de nuestra lista. Consulta los documentos de tu sistema de despliegue o servidor web para saber cómo hacerlo en tu situación.

Pero espera, ¿minificar y comprimir son lo mismo o diferente? En realidad, son un poco diferentes. Tanto minificar como comprimir reducen enormemente el tamaño de un archivo. Nosotros utilizamos la compresión. La minificación puede dar lugar a archivos ligeramente más pequeños -en parte porque eliminará los comentarios del código-, pero no es significativo. Los propios servidores web no admiten la minificación, pero si utilizas Cloudflare, hay una forma de activar la minificación automática. Probablemente no supondrá una gran diferencia, pero puedes probarlo.

## 4) Almacenamiento en caché de archivos a largo plazo

La cuarta y última cosa que tenemos que hacer es asegurarnos de que todos nuestros archivos estáticos están configurados para una caducidad a largo plazo. Como tenemos estos bonitos nombres de archivos versionados, cuando un usuario visite nuestro sitio, queremos que descargue este archivo una vez y nunca, nunca lo vuelva a descargar. Queremos que lo guarden en caché para siempre. Porque, si cambiamos algo dentro de este archivo, ¡cambiará todo el nombre del archivo! Y el navegador del usuario descargará naturalmente la nueva versión la próxima vez que visite el sitio.

Aquí, en "Cabeceras", vemos una cabecera de respuesta "Caduca". Por defecto, Platform.sh añade una cabecera "Caduca" para los activos estáticos, establecida en una hora. Podemos hacerlo mucho mejor.

En `.platform.app.yaml`, bajo `locations`... allá vamos... vemos...`expires: 1h`. Eso está bien como valor por defecto para otros activos estáticos que podamos tener en nuestro sitio. Así que voy a dejarlo así. Pero añade otra regla para ser más específico. Para la regex, si la URL coincide con `/assets/` cualquier cosa, entonces establece la cabecera `expires` en `365d`. Sí, 1 año - ¡eso es para siempre en tiempo de Internet!

[[[ code('3d2011b3a2') ]]]

¡Genial! Vamos a confirmarlo... ¡y desplegar, desplegar!

```terminal-silent
symfony deploy
```

Cuando termine, actualiza. No veremos nada obvio... pero comprobemos uno de nuestros archivos JavaScript, como `app.js`. Estamos buscando "Expires". Si no lo ves -o sigue sin caducar- haz un refresco forzado. Este es un caso raro en el que el archivo no ha cambiado, pero puede que aún tengamos la versión en caché de hace un minuto con la cabecera antigua. Y si ves esto, ¡enhorabuena! Ese es el objetivo.

Así pues, nuestros activos se están comprimiendo y tienen cabeceras "Caduca" a largo plazo. ¡Hemos marcado todas las casillas para tener un sitio con buen rendimiento!

A continuación: Vamos a demostrar que el sitio es rápido utilizando Lighthouse para perfilar el rendimiento del sitio. Aprenderemos cómo se descargan los archivos, cómo se construyen las páginas y haremos que nuestro frontend sea aún más eficiente.
