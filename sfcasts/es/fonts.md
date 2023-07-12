# Añadir fuentes

Otra necesidad común de CSS es una fuente personalizada. Mi fuente favorita de fuentes es https://fontsource.org, donde puedes buscar entre un enorme número de fuentes que tienen diversas licencias de código abierto.

Por ejemplo, una fuente popular es "Inter". Aquí, puedes descargar el archivo, y da unas instrucciones de instalación, que son interesantes: utiliza la fuente como un paquete `npm`.

Nosotros no usamos `npm`, pero podemos usar paquetes `npm`: y sabemos cómo.

Dirígete a jsDelivr para encontrarlo. Observa que el paquete se llama`@fontsource-variable/inter`. Voy a buscar `@fontsource/inter`. Y... al igual que con Bootstrap, ¡ahí está el archivo CSS! Para los frikis de las fuentes, si miraras dentro de este archivo, verías que se trata del peso 400, y es el archivo que utilizarías normalmente si lo instalaras a través de `npm` y lo importaras.

Copia esa URL y pégala en el navegador para ver qué aspecto tiene.

## ¿Fuentes variables?

Fíjate en que, en FontSource, recomiendan utilizar un paquete que empiece por `@fontsource-variable`. Las fuentes variables son geniales: en lugar de necesitar un archivo de fuente diferente para cada peso de fuente, como 400 frente a 800, una sola fuente variable puede contener todos los pesos, manteniendo un tamaño de archivo razonablemente pequeño. FontSource empezó a ofrecer fuentes variables hace poco.

Cambia la URL para utilizar `@fontsource-variable`. Esto es lo que realmente queremos. Cópialo, vuelve a `base.html.twig`, añade `<link rel="stylesheet">`, y pégalo.

[[[ code('5972b1a2f1') ]]]

Gracias a esto, podemos ir a `app.css`, dentro de la etiqueta `body`, y decir`font-family: 'Inter Variable'`, añadiendo `sans-serif` como copia de seguridad.

[[[ code('2106a43c03') ]]]

¡Vamos a comprobarlo! Fíjate bien en este texto. ¡Bum! Se ha actualizado gracias a esa nueva fuente.

Si te preguntas por qué no me limité a buscar el paquete `@fontsource-variable`en jsDelivr originalmente, es una pregunta justa: eso es lo que haría normalmente. jsDelivr es un espejo de todos los paquetes NPM. Sin embargo, debido a un error en la API de npmjs.com, ahora mismo, estos nuevos paquetes "variables" no se pueden encontrar en la búsqueda. Al parecer, el error se ha solucionado, así que esperemos que el problema desaparezca pronto.

La cuestión es que jsDelivr sí tiene el paquete que necesitamos, sólo que no podemos encontrarlo a través de la búsqueda. Es un poco molesto, pero debería ser temporal.

A continuación: Hagamos nuestro CSS un poco más elegante introduciendo Tailwind. Esto va a ser especialmente interesante porque Tailwind requiere un paso de compilación.
