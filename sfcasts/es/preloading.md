# Carga previa

Acabamos de descubrir un problema: nuestro navegador necesita descargar la página... y un archivo CSS antes de darse cuenta siquiera de que necesita descargar esta fuente. Puede que esto no sea un gran problema, pero hay una solución genial: la precarga.

## Añadir manualmente un enlace rel="preload"

Busca la URL de la fuente -es esta "normal"- y cópiala. A continuación, abre`base.html.twig` y, aquí arriba, añade una etiqueta `<link>`. A diferencia de un`<link rel="stylesheet"` normal que apunta a un archivo CSS, el propósito de esta etiqueta de enlace será gritar al navegador:

> Oye, aún no lo sabes, pero deberías descargar este archivo de fuentes.

Para ello, di `rel="preload"` y luego `href=""` y pega esa URL larga. Y cuando precarguemos fuentes, tenemos que añadir `as="font"`, `type="font/woff2"`, y`crossorigin=""` al final.

[[[ code('43068d15f9') ]]]

Vale, ¡a ver qué opina Lighthouse de esto! Después... seguimos puntuando 100 - ¡viva! -  y en "Evitar encadenar peticiones críticas", ese archivo de fuentes ha desaparecido.

## Precarga mediante un encabezado

Pero... ¿qué pasa con `app.tailwind.css`? El navegador descarga el HTML y ve inmediatamente la etiqueta `link` correspondiente. ¿Hay alguna forma de indicar al navegador que necesita descargar `app.tailwind.css` incluso antes de descargar el HTML? La respuesta, sorprendentemente, es ¡sí!

Pero necesitamos un componente de Symfony llamado "WebLink". En tu terminal, ejecuta:

```terminal
composer require symfony/web-link
```

Una vez hecho esto, de vuelta en `base.html.twig`, añade otra precarga aquí abajo de aspecto similar: `<link rel="preload" href="">`. Esta vez, utiliza una función Twig llamada `preload()` pasando la función normal `asset()` a apuntar a `styles/app.tailwind.css`. En este caso, esta función `preload` necesita otra opción llamada `as: 'style'`.

[[[ code('ecf6b6c52d') ]]]

Ya está Ve a actualizar la página y "Ver fuente de la página". No te sorprendas: se muestra una etiqueta `preload`. Y... ¡hasta ahora, la función `preload()` de Twig no parece más que un atajo semi inútil!

Es más, para el archivo `app.tailwind.css`, ¡esta etiqueta `link` no tiene sentido! Esto básicamente le dice al navegador

> Oye, deberías empezar a descargar este archivo CSS.

Pero... ¡una línea más tarde, lo habría encontrado de todos modos! Entonces, ¿por qué hicimos esto? Resulta que la función `preload()` hace dos cosas: emite la etiqueta`link` `href` ... pero también le dice a Symfony que debe añadir una cabecera `preload`a la respuesta.

Ve a las herramientas de red, selecciona "Todo", busca nuestra página principal, ve a "Encabezados", y mira en "Encabezados de respuesta" ¡Woh! ¡Tenemos una nueva cabecera "Enlace" llamada "precarga" que apunta a nuestro archivo CSS! Así, cuando el navegador empieza a descargar la respuesta, en la parte superior ve una indicación de que debe empezar a descargar ese archivo CSS

Si volvemos a Lighthouse y analizamos de nuevo la carga de la página... aquí abajo... ¡precioso! Toda esa sección ha desaparecido.

## precargar JavaScript en importmap.php

Hay algunas cosas más aquí, como "Mantén bajo el número de peticiones y pequeño el tamaño de las transferencias", pero no son realmente advertencias: sólo algo a tener en cuenta.

Pero sobre el tema de la precarga, aunque no tengamos más advertencias, hay otro punto en el que la precarga puede mejorar el rendimiento, y tiene que ver con JavaScript.

En `importmap.php`, hay una clave llamada `preload` de la que no hemos hablado. Está configurada en `true` para `app`. Ponla en `false`.

[[[ code('315e3713ee') ]]]

Ahora, muévete y ejecuta otro informe Lighthouse. Seguimos obteniendo una puntuación de 100, pero si bajamos aquí, ah: "Evitar encadenar peticiones críticas" ¡ha vuelto! ¡Y fíjate! Tenemos la página HTML, hasta `app.js`, luego `bootstrap.js`, el cargador de Stimulus, el propio Stimulus, los controladores... vaya. Se están encadenando un montón de cosas.

Es el mismo problema que vimos con los archivos CSS y de fuentes. Primero, nuestro navegador descarga el HTML. Luego ve que necesita descargar `app.js`. Una vez que descarga ese archivo, ve que necesita descargar `bootstrap.js`. Luego, se da cuenta de que necesita descargar el `stimulus-bundle/loader`, y así sucesivamente: uno por uno. En lugar de descargar todas esas cosas en paralelo, tiene que descubrirlas poco a poco.

`preload` arregla eso. Cámbialo de nuevo a `true`, actualiza la página y visualiza la fuente de la página.

[[[ code('6fd1a495d9') ]]]

Sabemos que la función Twig `importmap()` descarga `importmap` y`<script type="module">`. Pero también vuelca estas cosas de `modulepreload`. Estas son geniales. Como dijimos `'preload' => true` para `app`, añade un`<link rel="modulepreload">` para `app.js`. Eso indica al navegador que debe empezar a descargar `app.js` inmediatamente. Aunque, en realidad, eso no es importante porque, de todos modos, se habría dado cuenta en aproximadamente 1 microsegundo.

Lo realmente importante es que AssetMapper ve que `assets/app.js` importa`bootstrap.js`. Y como `app.js` está precargado, también precarga`bootstrap.js`. Y como éste importa `./lib/vinyl.js`, también precarga`./lib/vinyl.js`. Así que descargará los tres archivos inmediatamente.

En este punto, si ejecutáramos Lighthouse de nuevo, no se quejaría de ninguna de estas peticiones encadenadas. Pero aún podemos mejorar. En las herramientas de red, para JavaScript, fíjate en la columna de cascada. Vemos que empiezan a descargarse unos cuantos archivos, y luego unos cuantos más... y unos cuantos más. Así que seguimos teniendo este problema de encadenamiento, aunque aparentemente no es lo suficientemente grave como para que Lighthouse informe sobre él.

Sabemos que `bootstrap.js` se está precargando, pero`@symfony/stimulus-bundle` no... aunque se haya importado de`bootstrap.js`. ¿Por qué la precarga no "sigue" esa importación como las demás?

Lo que hay que entender es que, como estamos precargando `app.js`, cualquier cosa que `app.js` importe con una importación relativa también se precargará automáticamente. Pero cualquier cosa que importemos con una importación desnuda, como `lodash/camelCase` o`@symfony/stimulus-bundle`, no se precargará automáticamente. Tal vez deberían, pero tienen sus propias entradas dentro de `importmap.php`, así que tú controlas la precarga de esas independientemente.

En este punto, realmente estamos optimizando el rendimiento -quizás sobreoptimizando-. Pero si quieres evitar este problema de encadenamiento, podrías añadir `preload` a los elementos que sabemos que serán críticos para la representación de la página. Por ejemplo, `@hotwired/stimulus`es crítico, `stimulus-bundle` es crítico porque es lo que carga nuestros controladores, y `@hotwired/turbo` también es crítico.

[[[ code('52e792cb98') ]]]

Cuando actualizamos... nada cambia: sólo tenemos más elementos `modulepreload` en el HTML. Si ejecutamos Lighthouse una vez más, seguimos obteniendo una puntuación de 100, y puedes ver que no hay problemas importantes aquí abajo. Fantástico.

## Precarga todo

Por cierto, si ahora estás pensando:

> ¿Por qué no lo precargamos todo?

¡Es una buena idea! Pero, ¡no lo hagas! Tu navegador es inteligente, y sin precargas, tiene un algoritmo muy inteligente para determinar el mejor orden de descarga de los archivos para cargar las cosas lo más rápido posible. Si lo precargaras todo, probablemente el orden de carga no sería tan bueno. Utiliza las precargas sólo para los activos críticos.

¡De acuerdo! ¡Lo hemos conseguido! Creo que AssetMapper es un soplo de aire fresco, ¡y espero que tú pienses lo mismo! Hay algunas cosas que no hace, como agitar árboles o manejar TypeScript. Pero para un gran número de proyectos, ¡es una gran solución! Y lo mejor es que sigues escribiendo JavaScript normal. Así que si alguna vez necesitas cambiar a un sistema de compilación, puedes hacerlo.

Dinos lo que piensas, y esperamos poder hacer más mejoras para Symfony 6.4. Muy bien amigos, ¡hasta la próxima!
