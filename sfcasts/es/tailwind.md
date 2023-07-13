# CSS de Tailwind

El HTML de nuestro sitio ya tiene estilo con Tailwind: todas las clases que ves aquí proceden de él. Así que si conseguimos instalar Tailwind, deberíamos tener un sitio mucho menos feo.

Tailwind es interesante porque no es sólo un archivo CSS que incluyes: requiere un paso de compilación. Y eso está muy bien Que no tengamos un sistema de compilación para todo no significa que no podamos elegir añadir uno para algunas cosas concretas.

## Utilizar TailwindBundle

Antes de entrar en materia... más o menos una semana después de grabar esto, creamos un bundle que facilita enormemente la incorporación de Tailwind. Se llama, creativamente, [TailwindBundle](https://github.com/symfonycasts/tailwind-bundle)! Ver cómo se puede configurar un pequeño sistema de compilación sigue siendo interesante, pero si quieres saltarte este capítulo y dirigirte a ese bundle en su lugar, no herirás mis sentimientos. El bundle básicamente automatiza lo que vamos a hacer.

## Descargar el ejecutable independiente

Para que todo esto funcione, necesitamos el archivo binario de Tailwind. Como vemos aquí, podríamos instalarlo con Node... y esa es una opción realmente flexible. Tendrías un archivo `package.json`... pero en lugar de contener WebpackEncore y un montón de cosas más, sólo tendría Tailwind.

La otra opción, que evita por completo la necesidad de Node, es utilizar el ejecutable independiente. Haz clic en "Standalone CLI build" para ir a la página de lanzamiento de Tailwind. Busca la versión que necesitas: para mí, es "tailwind-macos-arm64". Puedes descargártela aquí, pero voy a copiar la dirección del enlace... para poder descargármela a lo grande mediante curl: `curl -slO` ¡y luego pega!

No importa dónde lo pongas, pero voy a moverlo al directorio `bin/`y renombrarlo a `tailwindcss`... en lugar de ese nombre tan largo. Por último, como otras máquinas -como los ordenadores de nuestros compañeros de trabajo o la máquina que despliega nuestro sitio- podrían necesitar una versión diferente de este archivo, vamos a ignorarlo.

[[[ code('f768a8db80') ]]]

Así que sí, esto significa que cada uno tendrá que descargar su propio binario de Tailwind.

El último paso es hacerlo ejecutable. En un sistema basado en Linux, eso es:

```terminal
chmod +x bin/tailwindcss
```

Ah, y hay un último paso adicional si estás en un Mac. Ejecuta:

```terminal
open bin/tailwindcss
```

Si es la primera vez que descargas el archivo, te pedirá que verifiques que quieres abrirlo desde el punto de vista de la seguridad.

## Inicializar Tailwind

Muy bien Ya tenemos el ejecutable `bin/tailwindcss`, que no requiere Node. A partir de aquí, podemos seguir la documentación normal. Esto es algo que me gusta mucho de la nueva filosofía del frontend. Si necesitas un sistema de compilación, puedes utilizar directamente el sistema de compilación de Tailwind y seguir sus instrucciones: no necesitas una solución específica para Symfony.

Aquí dice que tenemos que Ejecutar:

```terminal
tailwindcss init
```

Así que ¡hagámoslo!

```terminal
./bin/tailwindcss init
```

Esto crea un nuevo y brillante archivo `tailwind.config.js`. ¡Vamos a comprobarlo!

[[[ code('e5bbc32f38') ]]]

Lo más importante es configurar la clave `content`. Esto le dice a Tailwind dónde debe buscar HTML que pueda contener clases Tailwind. Busca su documentación específica para Symfony. ¡Aquí abajo tienen exactamente lo que queremos! Copia la clave `content`... ¡y pégala! Es decir... ¡pégala en el lugar correcto!

[[[ code('379f25cc80') ]]]

El último paso es copiar las tres líneas de la directiva base de Tailwind... y ponerlas dentro de `app.css`. Quitaré las cosas de Bootstrap... pero mantendré un poco de nuestro código personalizado aquí abajo. ¡Qué bien!

[[[ code('c2d106de6f') ]]]

## Creando el archivo CSS

Finalmente, ¡estamos listos para construir! En tu línea de comandos, ejecuta `bin/tailwind`, utiliza`-i` para apuntar al archivo de entrada `assets/styles/app.css`, y luego `-o` para indicarle dónde debe salir el código final. Utiliza `assets/styles/app.tailwind.css` para que esté en el mismo directorio, lo cual es importante para que las rutas relativas de las imágenes sigan funcionando. Al final, añade `-w` para que siga ejecutándose y vigilando los cambios:

```terminal-silent
./bin/tailwindcss -i assets/styles/app.css -o assets/styles/app.tailwind.css -w
```

¡Y ya está! ¡Construido! Por aquí, tenemos un archivo `app.tailwind.css` que contiene todas las cosas buenas. ¡Fantástico!

En `base.html.twig`, en lugar de apuntar a `app.css` -que ahora es una especie de archivo fuente "interno"- apunta a `app.tailwind.css`.

[[[ code('b5e761a82e') ]]]

Momento de la verdad. ¡Vuelve al navegador! Actualiza. ¡Nuestro sitio tiene estilo! Eso significa que podemos deshacernos de las cosas de Bootstrap: quitar el enlace Bootstrap CDN... ya que sólo estábamos demostrando cómo funciona... y también el botón de aquí abajo.

¡Así queda bien!

## Ignorar el archivo creado

Pero, ¿qué pasa con este archivo compilado de `app.tailwind.css`? ¿Lo ignoramos de git? ¿Lo confirmamos?  ¡Tú decides! Podemos confirmarlo, ya que facilitaría el despliegue, pero en general no queremos confirmar las cosas creadas. Yo lo ignoraré... y luego veremos cómo funciona en nuestro proceso de despliegue.

[[[ code('5b07168255') ]]]

Vale, ¡hecho! Siguiente: Pasemos a JavaScript.
