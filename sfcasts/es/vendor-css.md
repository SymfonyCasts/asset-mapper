# cSS de terceros

Hemos hablado de añadir CSS a nuestro sitio, pero ¿qué pasa con el CSS de terceros, como Bootstrap? Con un sistema de compilación como Encore, tenemos un archivo `package.json`, y podemos ejecutarlo:

```terminal skip-ci
npm install bootstrap
```

En AssetMapper, como no hay Node, no tenemos un sistema tan fácil para coger paquetes CSS. Pero aún podemos obtenerlos.

## Encontrar paquetes en jsDelivr

Me gusta utilizar jsDelivr para esto: un CDN para todos los paquetes NPM. Aunque al final no lo utilices como CDN, es una buena forma de encontrar y descargar lo que necesitas.

Busca "Bootstrap" y... ahí está. Muchas veces, encontrarás el archivo CSS que necesitas justo aquí arriba, así. Le daré a "Copiar HTML + ISR". Si no ves el archivo CSS aquí... o necesitas uno diferente, puedes hacer clic en la pestaña "Archivos" para examinar todo el paquete. Por ejemplo - `dist/css/` y luego lo que necesites.

Vale, sabemos que el CSS con AssetMapper es deliciosamente aburrido... así que vuelve al bloque `stylesheets` y, encima de `styles/app.css`, pega la nueva etiqueta`link`.

[[[ code('46fdf5251c') ]]]

Si quieres evitar usar el CDN, puedes descargar este archivo directamente en tu proyecto. Como no hay un sistema de paquetes como NPM, yo probablemente crearía un directorio `assets/vendor/` y pondría el archivo dentro de él. Luego confirmaría ese directorio `assets/vendor/` en Git para mantenerlo en tu proyecto y versionado. Cometer archivos de proveedor en tu proyecto no es asombroso, pero no es un gran problema y es tu mejor opción en este momento si quieres evitar un CDN. Me verás hacer esto más adelante para un archivo JavaScript.

Vale, ¡vamos a ver si esto funciona! Desplázate hasta la mitad de la página y añade un `button` con `btn btn-primary` para utilizar algunos estilos comunes de Bootstrap.

[[[ code('795e079dad') ]]]

Cuando vayamos al sitio y actualicemos... ¡funciona! ¡Precioso!

## ¿Bootstrap Sass?

Vale, pero ¿y si quiero modificar Bootstrap? El propio Bootstrap está construido con Sass. Así que, si quieres, puedes crear archivos Sass que anulen las variables de Bootstrap, por ejemplo para cambiar los colores por defecto.

Hay dos cosas importantes al respecto. En primer lugar, puedes utilizar Sass con AssetMapper. Hay detalles en la documentación sobre cómo hacerlo... y esperamos añadir pronto un bundle para hacerlo aún más fácil.

Además, en un momento, vamos a añadir Tailwind CSS a nuestro sitio, que no requiere Sass, pero tiene un flujo de trabajo muy similar porque Tailwind necesita ser "construido".

Y, si quieres utilizar Sass con Bootstrap, una forma sencilla de obtener el código fuente de Bootstrap es a través del paquete oficial de Composer para Bootstrap:

```terminal skip-ci
composer require twbs/bootstrap
```

Si esto es algo que quieres, consulta la documentación de AssetMapper.

## Variables CSS de Bootstrap

La segunda cosa importante es que, dependiendo de lo que quieras hacer, puede que no necesites utilizar Sass para personalizar Bootstrap. Esto se debe a que Bootstrap también expone variables CSS, aunque no son tan potentes.

Podemos ver esto más abajo en la sección "Personalizar" "Variables CSS". Las variables CSS son una función del navegador que te permite establecer variables dentro del CSS y luego hacer referencia a ellas. No es necesario usar Sass.

Por ejemplo, en `app.css`... en la parte superior... añade un pseudo selector `:root`, que es un lugar habitual para inicializar variables que se utilizarán más adelante. Aquí, anula una variable CSS que Bootstrap proporciona y utiliza: `--bs-border-radius`. Establécela en `1rem`.

[[[ code('10e0bc3030') ]]]

Esto debería hacer que los bordes sean notablemente más grandes. De vuelta al navegador... ¡funciona! El radio del borde es ahora mayor en todo el sitio. Esta es una de las variables que encontrarás en la documentación de Bootstrap.

Sin embargo, no siempre es tan sencillo. Digamos que queremos anular este color primario. Podrías pensar que puedes hacerlo buscando "primario"... aquí arriba... y anulando algo como `--bs-primary`. Eso es más o menos correcto.

Si inspeccionas nuestro botón, este color es el color de fondo real. Pero mira. Intenta anularlo cambiándolo a un color ligeramente más claro. Luego vuelve y pruébalo. No hace nada.

[[[ code('447b6893dd') ]]]

Copia la URL del CDN, ponla en tu navegador y quita el `.min` para que podamos ver qué está pasando. Encima, está configurando todas esas bonitas variables CSS. Busca `btn-primary`. No voy a profundizar demasiado, pero dentro de `.btn-primary`, está configurando estas variables CSS con estos colores codificados, en lugar de utilizar otras variables CSS que podemos controlar.

¿Y qué hacemos? En este caso, volvemos a la estrategia básica de anular CSS... aunque al menos podemos utilizar variables CSS cuando hacemos esto.

Vuelve a `app.css` y pegaré algo de estilo para `.btn-primary`. Esto anula las variables establecidas por Bootstrap con un color diferente. Al menos estamos utilizando la variable `bs-primary`: la establecemos aquí y podemos hacer referencia a ella en tantos sitios como queramos. Así pues, una anulación CSS bastante básica, pero con menos repeticiones.

[[[ code('490908f914') ]]]

Y cuando lo probamos... cambia el color. ¡Genial! Así que las variables CSS son una forma de personalizar Bootstrap, pero Sass sigue siendo una opción aún más potente.

A continuación: vamos a coger una cosa más de estilo externo: ¡una fuente de código abierto!