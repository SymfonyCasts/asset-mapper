# Configurar el despliegue de platform.sh

¡Acabamos de desplegar una versión semifuncionante de nuestro sitio en platform.sh! Todo lo que hizo falta fue un comando para arrancar unos cuantos archivos de configuración y otro para crear el proyecto dentro de platform.sh.

Pero... tuvimos algunos errores y advertencias por el camino. En la parte superior de la salida, vemos una advertencia sobre el uso de una versión antigua de Composer. En un minuto, veremos cómo y por qué se utiliza Composer durante el despliegue. Pero cuando lo hace, por alguna razón, está utilizando una versión antigua de Composer.

Afortunadamente, nos avisa y nos dice cómo solucionarlo.

## .platform.app.yaml y cómo funciona el despliegue

Copia esta línea `dependencies`. A continuación, abre `.platform.app.yaml`. Este es el archivo principal de despliegue: casi todos los ajustes de despliegue que harás se harán aquí.

El proceso de despliegue consta de dos pasos. El primero es el paso `build` en el que se construye tu código: puedes considerarlo como el paso que prepara todos los archivos físicos que necesita tu proyecto. Una vez finalizado el paso `build`, se crea un contenedor, se introducen los archivos y se ejecuta la segunda y última parte del proceso: el paso `deploy`. Aquí es donde puedes ejecutar algunos comandos finales.

¿Ves estos scripts `symfony-build` y `symfony-deploy`? Se trata de scripts prefabricados que contienen la mayor parte de lo que tu aplicación necesita para desplegarse. Si los descargaras y los abrieras, verías cosas como ejecutar `composer install`, calentar la caché y ejecutar migraciones de bases de datos. Podemos añadir cosas personalizadas por encima o por debajo.

La configuración tiene `mounts` -para directorios que quieras mantener persistentes entre despliegues- extensiones de PHP, tu versión de PHP, y bastantes cosas más.

## Utilizar una versión más reciente de Composer

En cualquier lugar del interior, pega la línea `dependencies`... y asegúrate de que no tiene sangría.

[[[ code('b1a9891dc6') ]]]

Y ya estamos utilizando la versión 2 de Composer.

## Configurar el servidor de base de datos

El segundo error que tuvimos, cerca de la parte inferior, fue,

> no se pudo encontrar el controlador

Esto viene de cuando el script `symfony-deploy` intenta ejecutar nuestras migraciones de base de datos. Localmente, estamos utilizando Postgres. Puedes verlo en `docker-compose.yml`. ¿Ya tenemos una base de datos en Platform.sh? La respuesta es... en realidad sí.

Además del archivo principal de despliegue - `.platform.app.yaml` - tenemos un directorio `/.platform`con algunos archivos más. El más importante es `services.yaml`. Aquí es donde definimos servicios como bases de datos, Redis, Elasticsearch y otros. Cuando inicializamos el proyecto, ¡se dio cuenta de que estamos utilizando Postgres y nos añadió una base de datos!

[[[ code('e2d3109128') ]]]

El error que obtenemos no es porque no pueda encontrar la base de datos: ¡es porque a nuestra instalación de PHP le falta el controlador `PDO_PGSQL`! Y gracias a `.platform.app.yaml`, añadirlo es fácil.

Busca `extensions`, y añade `pdo_pgsql`.

[[[ code('a31d9791f2') ]]]

Vale, ¿listo para volver a desplegar? Recuerda: el despliegue se realiza mediante un push, así que tenemos que confirmar estos archivos. Ejecuta `git commit -m` con un mensaje inspirador.

```terminal-silent
git commit -m "tweaking deploy script"
```

Ahora ejecuta:

```terminal
symfony deploy
```

Esto ejecuta los mismos pasos que nuestro primer despliegue, que ahora entendemos que incluyen un paso`build` y luego un paso `deploy`. Tardará uno o dos minutos, pero debería ser un poco más rápido porque no necesita volver a aprovisionar el certificado SSL.

## Ver los Registros

Al final... el comando de migración seguía fallando, pero con un error diferente:

> Conexión denegada

Ah, ignora eso por un momento. En lugar de eso, vuelve al sitio y actualízalo. La página de inicio sigue funcionando. Pero... eso es porque nuestra página de inicio no utiliza la base de datos. Si pulsas "Examinar mezclas"... ¡error 500! Ese error 500 se debe probablemente a un problema de conexión con la base de datos. Pero vamos a suponer que no tenemos ni idea de cuál es la causa. ¿Cómo podríamos averiguarlo? Aquí es donde el comando

```terminal
symfony logs
```

es muy útil. Se conecta al "entorno" -o "servidor"- de platform.sh al que esté conectada nuestra rama git actual y devuelve la información del registro. Hay un montón de opciones, pero pulsa `2` para ir al registro de la "aplicación". Esto representa los registros de Symfony procedentes de nuestra aplicación. Y... oooh. Veo varios:

> Conexión al servidor [..] fallida: Conexión denegada

## Añadir la "relación" de base de datos

Reflexionemos sobre esto. Parece que tenemos Postgres configurado, gracias al archivo`services.yaml`. Pero nunca hemos configurado nuestra aplicación para que hable con ella. Recuerda que en `.env`, tenemos una variable de entorno `DATABASE_URL` que se supone que apunta a nuestra base de datos. Nunca la configuramos en nuestro sitio de producción, así que sólo utiliza este valor por defecto. Y no es de extrañar que no funcione.

¿Cómo podemos configurar `DATABASE_URL` para que apunte a... dondequiera que esté este servidor de base de datos? La respuesta es... nosotros... eh... ¿no? Y es bastante guay.

Platform.sh tiene esta idea de las relaciones. Tienes una serie de servicios en`services.yaml`. Pero tu aplicación no puede hablar con ellos hasta que los vincules mediante lo que se llama una "relación".

Busca "relaciones" en `.platform.app.yaml`. Todavía no está aquí, así que vamos a añadirla. Cada "relación" tiene un nombre interno. Técnicamente podría ser cualquier cosa, pero, en la práctica, deberías utilizar `database`. Veremos el significado de esto dentro de un momento. Ponlo en la palabra `database`, porque es la clave que tenemos aquí, y luego `:` seguido del tipo de servicio, que es `postgresql`.

[[[ code('d2e99c869e') ]]]

Esta sintaxis siempre me ha parecido rara. Lo importante es que la clave podría ser cualquier cosa, como `banana`, pero este `database` hace referencia a este `database`en `services.yaml`, y `postgresql` hace referencia a este `postgresql`.

Pero aunque la primera clave `database` podría ser cualquier cosa, he utilizado `database` a propósito. Symfony hace una cosa muy buena cuando despliega a través de Platform.sh. Ve esta relación, se da cuenta de que es para una base de datos, ¡y expone automáticamente una variable de entorno que contiene la información de conexión a esa base de datos!

¿Cómo se llama esta variable de entorno? Como hemos utilizado la clave "base de datos", se llamará `DATABASE_URL`. En otras palabras, va a establecer esta variable de entorno por nosotros. ¡Voy a probarlo!

## SSH en el Contenedor

Una de mis cosas favoritas de Platform.sh es que puedes conectarte por SSH a tu contenedor. Observa:

```terminal
symfony ssh
```

Allá vamos. Una vez aquí, si quieres ver todas las variables de entorno, ejecuta:

```terminal
printenv
```

¡Mira eso! No verás nada que empiece por "base de datos", pero deberíamos después de desplegar este próximo cambio. Escribe `exit`, ejecuta

```terminal
git status
```

y luego

```terminal
git add -p
```

¡Eso es lo que queremos! Confirma con

```terminal
git commit -m "adding database relation"
```

Y

```terminal
symfony deploy
```

Esta vez, se despliega mucho más rápido. Como no hemos cambiado ningún código de la aplicación, platform.sh ha sido lo suficientemente inteligente como para utilizar el código de nuestra antigua aplicación, en lugar de volver a hacer toda esa construcción. Podemos verlo:

> Reutilizando la compilación existente para este ID de árbol

Y ¡hey! ¡Esta vez, vemos que `Successfully migrated`! Ejecuta nuestra migración sin problemas. Cuando giramos y comprobamos el sitio... funciona. Aún le faltan todos nuestros estilos... pero eso lo arreglaremos a continuación. Lo importante es que la base de datos funciona.

Puedes ver la diferencia si ejecutas

```terminal
symfony ssh
```

y

```terminal
printenv
```

Esta vez, hay varias variables `DATABASE_`, incluida la más importante`DATABASE_URL`.

Vale, la última pieza que le falta a nuestro sitio desplegado es... ¡todos sus activos! Veamos ahora qué se necesita para desplegar un sitio AssetMapper.
