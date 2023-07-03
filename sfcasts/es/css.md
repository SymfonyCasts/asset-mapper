# CSS e imágenes de fondo

Cuando hablamos del frontend de un sitio, nos referimos principalmente a dos cosas: CSS y JavaScript. Empecemos por la parte CSS... que es muy sencilla con AssetMapper. Crea un archivo CSS dentro del directorio `assets/` e inclúyelo con una etiqueta `link` a la antigua usanza que utilice la ruta lógica del archivo. Y ya está. Cero magia.

Esto es un poco diferente de Encore. Con un sistema de compilación como Encore, puede que estés familiarizado con hacer cosas como esta: `import './styles/app.css`. Ese tipo de cosas no funcionan en un entorno de navegador. Las declaraciones de importación son para importar archivos JavaScript, punto. De acuerdo, técnicamente puedes cargar CSS de forma perezosa, pero ése es un caso extremo del que no necesitamos preocuparnos ahora.

La cuestión es que no puedes importar archivos CSS desde archivos JavaScript y no pasa nada: añadir una etiqueta `link` funciona de maravilla.

## Referenciar imágenes desde dentro de archivos CSS

Vale: ya sabemos que podemos hacer referencia a cualquier archivo del directorio `assets/` utilizando la función`asset()`... lo que ya hemos hecho dos veces.

Pero, ¿y si necesitamos hacer referencia a un archivo -como esta imagen- desde dentro de un archivo CSS?

[[[ code('de3f8d3200') ]]]

Echa un vistazo. Aquí arriba, tenemos un pequeño icono de registro en la esquina superior izquierda. Cámbialo por un `span` con `class="bg-logo"` para que podamos incluir nuestra imagen del pingüino en su lugar. Copia ese encabezado de clase `bg-logo` en `app.css`, añade `.bg-logo` y... añadiré algunos estilos básicos.

[[[ code('17c2fac770') ]]]

La gran pregunta es: ¿cómo podemos establecer la imagen de fondo... ya que el`penguin.png` final tendrá un nombre de archivo versionado? La respuesta es: exactamente como lo harías normalmente: `url()` y luego la ruta relativa al archivo: `../images/penguin.png`.

[[[ code('62ff749566') ]]]

Así es exactamente como se hace en Encore y exactamente como se haría si estos archivos se sirvieran directamente a nuestro navegador. Simplemente tenemos que escribir el código "correcto" y funcionará.

Añadamos 2 estilos más para el fondo... ¡y a probar! Actualiza y... ¡sí, funciona! Inspecciona la imagen... y luego mira el archivo CSS final. Abrámoslo en una nueva pestaña.

¡Perfecto! En su mayor parte, los archivos finales coinciden exactamente con los archivos de origen. Sin magia. Sin embargo, en este caso, AssetMapper hizo un pequeño cambio. En el archivo original, nos referíamos a `../images/penguin.png`. Pero aquí tenemos`../images/penguin-versionhash.png`. Sí, AssetMapper hizo ese pequeño cambio para que las cosas siguieran funcionando, a pesar de los nombres de archivo versionados.

La cuestión es: puedes codificar como si nada... y todo funciona.

A continuación: ¡invitemos a la fiesta a CSS de terceros como Bootstrap y fuentes!
