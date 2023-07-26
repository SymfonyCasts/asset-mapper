# Controladores de Stimulus perezosos

Esto se está liando: déjame cerrar unos cuantos archivos... y luego abrir`assets/controllers/goodbye-controller.js`. Imagina, por un momento, que este controlador es enorme. O, más probablemente, que importa un gran paquete de terceros como `d3` para gráficos. Pero, sólo vamos a utilizar este controlador en algunas páginas.

Esto es lo que pasa. Para registrar tus controladores con Stimulus, todos estos archivos se descargan inmediatamente. Así que se carga la página, se inicia Stimulus, se descargan todos estos archivos y también se descargan todos los archivos que importan. Eso suele estar bien, pero si estás importando algo grande, puede ser un desperdicio.

Para solucionarlo, encima de la clase, puedes añadir una sintaxis muy especial:`/* stimulusFetch: 'lazy' */`.

[[[ code('3b03f096b8') ]]]

Esto funciona gracias a StimulusBundle. Cuando detecta esto, le dice a Stimulus que se contenga y no descargue este archivo JavaScript ni nada de lo que importe hasta que haya un elemento que coincida con esto en la página.

Observa. Antes de hacer ese cambio, si buscábamos "adiós", se cargaba ese controlador, aunque no se utilizara en esta página. Pero ahora, actualiza y busca "adiós". ¡No está ahí! Inspecciona el elemento`data-controller="hello"`. Cámbialo por `goodbye` y... ¡boom! Funciona! Puedes ver que se ha activado (eso es lo que hace nuestro `Goodbye controller!` ), y si miramos en la pestaña Red, ahora se ha descargado. Me encanta esta función.

Esto también se puede hacer para paquetes de terceros. Si miras en`assets/controllers.json`... Turbo no es un buen ejemplo de esto, pero si dijéramos `"fetch": "lazy"` en cualquiera de ellos, tendrían el mismo comportamiento que acabamos de ver.

[[[ code('b292bac1cf') ]]]

¡Ya está! ¡El capítulo más fácil! Utilízalo para aligerar tu página inicial si tienes algunos controladores Stimulus pesados que sólo se utilizan en determinadas páginas.

Siguiente: a veces, suspiro profundo, los dioses de la tecnología nos fruncen el ceño y las cosas no funcionan. Aprendamos algunos trucos para ayudar a depurar cuando eso ocurra.
