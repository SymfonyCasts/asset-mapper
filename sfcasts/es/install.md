# Instalación de AssetMapper

Ya sabemos que podemos ejecutar JavaScript moderno directamente en nuestro navegador. Pero para facilitar el proceso, vamos a instalar un nuevo componente de Symfony llamado AssetMapper.

Busca tu terminal y ejecuta:

```terminal
composer require symfony/asset-mapper symfony/asset
```

Incluyo el segundo paquete porque nos proporciona esa bonita función `asset()` en Twig. Ya está instalado en este proyecto - asegúrate de que lo tienes en el tuyo.

Antes de empezar: AssetMapper es experimental en Symfony 6.3, por lo que es probable que haya algunas interrupciones de retrocompatibilidad antes de la 6.4. Pero como veremos, los conceptos son sólidos, y puedes desplegar un sitio super-performante con AssetMapper hoy mismo.

## Cambios de la receta Flex

Ejecuta:

```terminal
git status
```

Oooh: la receta Flex para AssetMapper ha añadido varias cosas. En primer lugar, nos ha proporcionado un directorio `assets/`... que es prácticamente idéntico al que tendrías si instalaras WebpackEncore. Tenemos un archivo `app.js` -este será el principal, el que se ejecute- y también `app.css`: el archivo CSS principal.

[[[ code('68df41237d') ]]]

[[[ code('9031ca801a') ]]]

En `templates/base.html.twig`, la receta también añadió una etiqueta `link` para apuntar a`app.css`. Hablaremos más sobre hojas de estilo más adelante, pero ya puedes ver que la configuración CSS es perfectamente sencilla.

[[[ code('d44f1d1210') ]]]

La receta añadió otra línea importante a este archivo: `{{ importmap() }}`. Que se asocia con un nuevo archivo `importmap.php`. Estos son importantes, y entraremos en detalle sobre ellos en breve.

Las conclusiones son que la receta creó unos cuantos archivos en `assets/` y añadió una etiqueta`link` a `base.html.twig`. Pero por lo demás, aún no ha pasado gran cosa.

## AssetMapper "Rutas"

Volviendo al terminal, la receta también ha creado un nuevo archivo `asset_mapper.yaml`. Abrámoslo: `config/packages/asset_mapper.yaml`.

[[[ code('abd34ac304') ]]]

AssetMapper tiene un concepto principal: lo apuntas a un directorio o conjunto de directorios, como `assets/`, y hace que todos los archivos de su interior estén disponibles públicamente, como si vivieran en el directorio `public/`. Veremos cómo se consigue en un minuto.

Pero antes de hacer nada, actualiza la página y... ¡el fondo se ha vuelto azul! Eso viene del archivo `app.css`. Y, en el registro de la consola, vemos un mensaje que procede de `assets/app.js`. Así que, de algún modo, mágicamente, sólo con ejecutar un comando`composer require`, estos dos archivos ya están expuestos públicamente y se están cargando en la página. A continuación, vamos a ver cómo funciona todo esto.