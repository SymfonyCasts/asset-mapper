# ¿Un mundo sin sistemas de construcción?

¡Hola! Bienvenidos a mi laboratorio de frontend, donde vamos a hacer algo que, sinceramente, pensé que nunca volvería a hacer. ¡Algo atrevido! Algo... quizá un poco loco. Vamos a escribir un frontend moderno con cero sistemas de compilación.

## Cómo hemos llegado hasta aquí

¡Hora de la historia! hace 7 años hablaba de cómo el JavaScript moderno requiere un sistema de compilación. Gritaba al mundo que teníamos que pasar de crear archivos JavaScript y CSS en un directorio "público" a construirlos con un sistema, como Webpack o Vite.

Estos sistemas de compilación se crearon porque los navegadores no admitían las funciones modernas que queríamos utilizar. Me refiero a la declaración `import`, `const`, la sintaxis de clases, etc. Si hubieras intentado ejecutar este tipo de JavaScript en un navegador, habrías sido recibido con tristes mensajes de error.

Por tanto, el sistema de compilación transpilaría (que es una palabra elegante para "convertir") ese JavaScript de aspecto nuevo a JavaScript de aspecto antiguo, para que pudiera ejecutarse en el navegador. También combinaría los archivos JavaScript y CSS, para que tuviéramos menos peticiones, podría crear nombres de archivo versionados, procesar TypeScript y JSX, Sass, y mucho más.

Estos sistemas son increíblemente potentes. Pero también añaden complejidad y pueden ralentizar la codificación. Así que estoy aquí, 7 años después, para decir que... ¡puede que ya no necesitemos esos sistemas de compilación! En este tutorial, vamos a escribir todo el JavaScript moderno que conocemos y amamos... pero sin ningún sistema de compilación ni Node. Sólo tú y el navegador: tal y como lo concibieron los Dioses de Internet.

## ¿Es esto para todos los proyectos?

Ahora, lo admito, hacer esto no será la mejor opción para todos los proyectos. Si quieres usar TypeScript, o estás usando React, Vue o Next.js, probablemente seguirás queriendo un sistema de compilación... y probablemente deberías usar su sistema de compilación. Omitir un sistema de compilación también significa que no hay agitación automática del árbol -si sabes y te importa eso-, aunque aprenderemos cómo puede seguir funcionando.

En su mayor parte, la programación con y sin sistema de compilación es idéntica, pero señalaré las pequeñas diferencias a lo largo del proceso. Y si te estás preguntando por cosas como los preprocesadores Sass o Tailwind, puedes utilizarlos y veremos cómo. El sitio final también va a ser tan eficiente y rápido como uno construido con un sistema de compilación.

## Configuración del proyecto

Bien, ¡manos a la obra! Codificar sin un sistema de compilación es una gozada: no necesitas nodos ni pilas. Así que deberías descargarte el código del curso de esta página y codificar conmigo. Después de descomprimir ese archivo, tendrás un directorio `start/`con el mismo código que ves aquí. Abre este archivo `README.md`. Como de costumbre, contiene todos los detalles de configuración que necesitarás. Yo ya he hecho la mayoría. El último paso es buscar tu terminal, entrar en el proyecto y ejecutar:

```terminal
symfony serve -d
```

para utilizar el binario `symfony` para iniciar el servidor web incorporado. Mantendré pulsado "comando", haré clic y... ¡hola Vinilo Mixto! Pero vaya que esta cosa tiene un aspecto raro y feo.

Se trata de un proyecto Symfony 6.3 -el mismo proyecto que hemos construido en la serie Symfony-. Tiene Doctrine instalado... pero no tiene nada particularmente especial, y ahora mismo, tiene literalmente cero CSS y JavaScript. No hay ningún directorio `assets/` ni nada escondido dentro del directorio `public/`.

Lo primero que quiero explorar es la realidad de que nuestro navegador puede manejar más cosas modernas de las que nos imaginamos... ciertamente mucho más de lo que yo me imaginaba hace unos meses. Veamos de qué va todo este revuelo probando nuestro navegador con un JavaScript moderno.
