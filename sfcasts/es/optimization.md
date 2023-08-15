# Optimización y perfiles

En lugar de decirte que el sitio es rápido, ¡vamos a probarlo! En Chrome, hay una herramienta llamada "Lighthouse", que también puedes conseguir para algunos otros navegadores. Ejecútala sólo para el rendimiento y selecciona "Analizar carga de página".

Esta es la mejor manera de ver si tienes algún problema de rendimiento en el frontend. Es probable que nuestra puntuación sea bastante alta -simplemente porque nuestro sitio es pequeño y rápido-, pero podemos utilizar el informe para detectar algunos posibles problemas. Y... ¡sí! Obtuvimos un 98 ¡sin sistema de construcción! Es asombroso. Pero podemos hacerlo aún mejor.

## Eliminar los recursos que bloquean el renderizado

Si nos desplazamos hacia abajo, podemos ver dónde están nuestras áreas problemáticas. La primera es "Eliminar recursos que bloquean la renderización", que apunta a nuestro archivo de fuentes. Gran parte de lo que vamos a hablar no tiene nada que ver con AssetMapper: se trata simplemente del rendimiento del frontend en general. Si abres `templates/base.html.twig`, tenemos una etiqueta `<link>` que apunta a este archivo de fuentes. 

[[[ code('0ed18657ba') ]]]

Cuando tu sitio ve una etiqueta `<link rel="stylesheet">, la descarga antes de renderizar la página. Así que básicamente congela el renderizado de la página hasta que finaliza la descarga.

Pero esto es interesante. Abre ese archivo... y tomemos una versión no minificada. Tiene un montón de posibles fuentes. Así es como funciona: nuestro navegador descarga este archivo inmediatamente... pero los archivos de fuentes en sí no se descargarán hasta que y a menos que utilicemos esta fuente. Además, `font-display: swap` le dice al navegador:

> Oye, no pasa nada por renderizar un texto que se supone que utiliza esta fuente, aunque la
> fuente aún no se ha descargado. Puedes utilizar primero la fuente predeterminada del sistema, mostrar el
> texto, terminar de descargar este archivo de fuente, y luego utilizarlo.

Esencialmente, este archivo CSS está escrito de forma que todos estos archivos de fuentes se descargarán perezosamente. El problema, que en realidad no es un gran problema, es que, en este punto, nuestro navegador sólo ve un archivo CSS y piensa:

> Necesito descargar ese archivo CSS ahora mismo y no puedo renderizar la página hasta que
> ¡hasta que termine!

Una vez que por fin ve el contenido CSS, descubre que hay un montón de archivos de fuentes que puede descargar perezosamente.

Así pues, los archivos CSS son recursos que bloquean la renderización... lo que normalmente es genial, porque no queremos que la página se renderice sin estilo durante medio segundo antes de que se descargue el CSS. Pero este archivo en concreto es curioso porque es un recurso que "bloquea la renderización"... pero no contiene nada crítico.

Si nos preocupamos lo suficiente como para eliminar este recurso que bloquea la renderización, podemos moverlo a `app.css`. Empieza copiando este archivo... o en realidad sólo las fuentes que necesitamos: muchas de ellas son para idiomas que no estamos utilizando. Yo copiaré las dos fuentes latinas... aunque es probable que ni siquiera necesitemos esta de extensión latina. A continuación, borra este archivo CSS por completo, ve a `assets/styles/app.css`, y pégalo. No son URL reales... así que copia la URL... pégala, quita el `index.css`... y debería estar bien. Vuelve a copiar la URL... y haz lo mismo aquí abajo.

[[[ code('55741db20b') ]]]

Perfecto. Esto añade cierta complejidad a nuestro código a cambio de una pequeña ganancia, así que yo diría que es una prioridad menor. Tenemos básicamente la misma cantidad de CSS que antes, pero hemos eliminado un pequeño recurso de bloqueo innecesario.

El otro fallo que tenemos es similar. Es para FontAwesome - concretamente, este archivo JavaScript. También está en `base.html.twig`. Como esta etiqueta `<script>` no tiene `defer` ni `async`, también bloqueará la renderización de la página. Si queremos, podemos añadirle `defer`, que dice:

> Comienza a descargar esto inmediatamente, pero no bloquees la página mientras está terminando.

[[[ code('4bd1fba315') ]]]

Como esto es para fuentes FontAwesome, en el peor de los casos la página se carga y nuestros iconos de fuentes aparecen un momento después.

## ¡Perfilar de nuevo!

Bien, ahora que hemos cambiado un par de cosas, vamos a probarlo. Para ahorrar tiempo de redistribución, volveré a mi sitio local y ejecutaré Lighthouse de nuevo. "Analizar carga de página"... haz esto un poco más grande, y... ¡fantástico! ¡Estamos obteniendo 100 localmente!

Pero si miras aquí abajo... todavía tenemos algunas oportunidades de mejora. Vemos "Servir imágenes en formatos next gen", que es algo bueno para comprobar más adelante, pero que no está relacionado con Symfony ni con AssetMapper. Esto de "Evitar servir JavaScript heredado a navegadores modernos" creo que se refiere al shim `importmap`: el código que hace que el mapa de importación funcione en todos los navegadores. Es pequeño y necesario, así que no es gran cosa.

## Evitar encadenar peticiones críticas

Pero debajo vemos "Evitar encadenar peticiones críticas". Este es probablemente el punto más importante de esta lista.

Esto es lo que ocurre. Como puedes ver, primero descarga el HTML. Una vez que lo hace, se da cuenta de que necesita descargar este archivo CSS. Una vez que descarga el archivo CSS, se da cuenta de que necesita descargar este archivo de fuentes. ¿Ves el problema? En lugar de saber -desde el principio- que los necesita y descargarlos todos a la vez en paralelo, nuestro navegador los va descubriendo poco a poco. En última instancia, esto significa que este archivo de fuentes tardará más en cargarse porque no puede empezar a descargarlo hasta que descargue otros archivos.

¿Cómo podemos solucionar esto? Precargando. Hablemos de este importante tema a continuación.
