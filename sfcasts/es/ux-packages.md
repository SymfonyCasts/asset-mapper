# Paquetes Symfony UX Stimulus

Ahora podemos crear controladores Stimulus personalizados con facilidad. La otra mitad de StimulusBundle es la posibilidad de obtener más controladores Stimulus gratuitos instalando un paquete UX. ¡Añadamos uno y veamos cómo funciona!

## Instalar Turbo

Empecemos por añadir Turbo. En tu terminal, di:

```terminal
composer require symfony/ux-turbo
```

Ésta es la parte más jugosa: al igual que cuando añadimos Stimulus, no necesitas hacer absolutamente nada más para configurarlo. Actualiza y... ¡simplemente funciona! Turbo elimina la necesidad de actualizar toda la página. Ve a tus herramientas de Red y haz clic en "Fetch/XHR". Despejemos esto para poder verlo todo. Perfecto. Entonces, si hacemos clic aquí arriba... ¡podrás ver que esto procede de una llamada AJAX! Sí, ya no se actualiza la página. Así que Turbo simplemente funciona. No hay ningún sistema de construcción que se interponga, y eso es hermoso.

## Los paquetes UX suelen añadir entradas Importmap

En la práctica, esto funciona porque se carga un nuevo archivo JavaScript llamado`turbo_controller.js`. Filtra las llamadas de red a JavaScript... y actualiza, porque las he borrado. ¡Ya está! Nuestra página carga `turbo_controller.js` y eso importa `@hotwired/turbo`, que inicia Turbo.

Abre `importmap.php`. Cuando instalamos el paquete UX Turbo, su receta añadió esta nueva entrada `@hotwired/turbo`. 

[[[ code('377be8f49c') ]]]

Este es un patrón muy común con los paquetes UX: si un paquete UX depende de un paquete de terceros, su receta añadirá ese paquete a tu `importmap` automáticamente. 
El resultado es que, cuando se hace referencia a ese paquete -como `import '@hotwired/turbo'` -, simplemente funciona.

## Cómo se cargan los controladores UX

La verdadera pregunta es: ¿quién carga `turbo_controller.js`, que vive en las profundidades del paquete PHP `symfony/ux-turbo`?

La respuesta es: el mismo truco que aprendimos hace un momento. Busca `controllers` y abre ese archivo en una pestaña nueva. Éste es el archivo dinámico que construye StimulusBundle. Resulta que busca paquetes en nuestro directorio `assets/controllers/`, que son estos dos, y lee el archivo `assets/controllers.json`. Cuando instalamos UX Turbo, añadió esta nueva sección aquí, que es donde activamos los distintos controladores. Activó uno llamado `turbo-core` con `"enabled": true` y añadió otro desactivado con `"enabled": false`. Así que cuando se construye este archivo, analiza el archivo `assets/controllers.json`, encuentra los controladores que hemos activado y los añade aquí.

[[[ code('5c4957130d') ]]]

El resultado final es que importa ese archivo de controladores aquí y lo exporta para que el archivo `loader.js` pueda registrarlo en Stimulus. Así, cualquier controlador en`assets/controllers/` o en este archivo se registra automáticamente.

## autoimportación y archivos CSS

Vuelve a `base.html.twig`. Cuando instalamos StimulusBundle, su receta venía con regalos, uno de los cuales era este `ux_controller_link_tags()`. Ahora mismo, eso no hace nada. Sin embargo, algunos paquetes UX vienen con archivos CSS. Los encontrarás bajo una clave llamada `autoimport`, que la receta añadirá bajo el controlador. Este `ux_controller_link_tags()` encuentra todos los archivos CSS de todos los controladores que tengas activados, y los emite. Nada demasiado sofisticado.

A continuación: vamos a aprender una cosa más sobre Stimulus, que resulta ser una de mis cosas favoritas: cómo hacer que nuestros controladores sean perezosos.
