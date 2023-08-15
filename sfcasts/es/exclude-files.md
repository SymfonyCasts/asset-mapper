# Excluir archivos

Ahora tenemos CSS -que estamos construyendo con Tailwind-, tenemos JavaScript, estamos incorporando JavaScript de terceros y estamos utilizando sintaxis JavaScript moderna. ¡Nuestra aplicación tiene todo lo que tiene una aplicación real! Claro, es un poco pequeña, pero estamos casi listos para desplegarla.

## Comprobación de los archivos expuestos

Antes de hacerlo, hagamos una rápida auditoría de los activos que están dentro de AssetMapper. Busca tu terminal y ejecuta:

```terminal
php bin/console debug:asset
```

Esto enumera todas nuestras rutas de activos, que incluyen nuestra ruta de activos principal - `assets/` - más algunas de bundles que han expuesto sus propios directorios. A continuación se muestra una lista de todos los archivos que se expondrán públicamente.

Ejecutamos este comando para ver si hay algo en esta lista que no queramos exponer públicamente. Por ejemplo, este archivo `assets/styles/app.css`. Esto es realmente un archivo fuente: no está pensado para que el usuario lo descargue directamente. Estamos utilizando Tailwind para incorporarlo a `app.tailwind.css`, y eso es lo que el usuario descargará. No es un gran problema que esté disponible públicamente, pero es un buen ejemplo de cómo podemos ocultar archivos "fuente" que no queremos exponer.

## Configuración del Mapeador de Activos

Empieza ejecutando

```terminal
php bin/console config:dump framework asset_mapper
```

Estamos pidiendo al sistema que nos dé ejemplos de configuración para todo lo que se puede configurar en `framework`, `asset_mapper`. Cuando instalamos AssetMapper por primera vez, su receta nos dio un archivo `config/packages/asset_mapper.yaml`. Aquí tenemos`framework`, `asset_mapper`, y una clave llamada `paths`. Cuando ejecutamos este comando... efectivamente, aquí arriba aparece `paths`. Debajo, tenemos otras cosas interesantes.

La primera es `excluded_patterns`. Así es como vamos a ocultar determinados archivos o rutas, y hablaremos más de ello en un minuto. También puedes controlar el`public_prefix`, que es a donde salen tus archivos en el directorio `public/`.

Este `extensions` no es superimportante... es sobre todo para el entorno `dev`... y hay algunas cosas más como tu `importmap_path`, e incluso algunos atributos que puedes poner en la etiqueta `<script>` que es volcada por la función `importmap()`.

## Excluir archivos / patrones

Hay algunas cosas buenas aquí... pero no tendrás que preocuparte de la mayoría de ellas, aparte de `excluded_patterns`.

Copia esa clave, gira hasta `asset_mapper.yaml`, y en el mismo nivel que `paths`, pega. Queremos excluir `assets/styles/app.css`.

[[[ code('07a53d9490') ]]]

Pero esto no es del todo correcto. Para comprobarlo, ejecuta

```terminal
php bin/console debug:asset
```

de nuevo. Si miras hacia arriba... ¡ `assets/styles/app.css` sigue ahí! Eso es porque`excluded_patterns` debe ser un glob. En otras palabras, cámbialo por`*/assets/styles/app.css`... y rodéalo de comillas.

[[[ code('42f5f44348') ]]]

Esto significa que se ignorará cualquier "ruta del sistema de archivos" que termine en `/assets/styles/app.css`. Y cuando volvamos a probar el comando...

```terminal-silent
php bin/console debug:asset
```

Genial. Esto es lo que queremos ver. Todos los archivos que estén aquí se volcarán en el directorio`/public/assets`. El hecho de que `assets/styles/app.css` no esté aquí significa que no se volcará en el directorio `public/`.

¡Creo que es hora de desplegar nuestro sitio! Vamos a configurar un despliegue a continuación en plataforma.sh.
