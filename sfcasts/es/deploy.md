# Despliegue en Platform.sh

Tengo una idea descabellada. Despleguemos este sitio de verdad.

## Requisitos para el despliegue de AssetMapper

Puedes desplegar tu código como quieras... utilizando cualquier servicio o servidor web. No importa con AssetMapper. El único requisito es que tu servidor web soporte HTTP/2 para que nuestros activos -los archivos JavaScript y CSS- puedan descargarse en paralelo superrápido. HTTP/2 es la razón por la que no es tan importante que nuestros archivos no se combinen para minimizar las peticiones.

Todos los servidores web - nginx, Caddy, lo que sea - soportan HTTP/2. O podrías añadir Cloudflare o un servicio similar delante de tu sitio, que te proporciona esto gratis... junto con algunas otras ventajas.

## configuración del archivo de configuración platform.sh

De todos modos, vamos a desplegar con Platform.sh, que es una "Plataforma como Servicio", lo que significa que podemos desplegar simplemente creando unos cuantos archivos de configuración. Y esta primera sección trata sobre cómo configurarlos. Una vez que hayamos terminado, hablaremos de algunos aspectos específicos del despliegue con AssetMapper.

Así que, ¡empecemos! Vamos a hacer la mayor parte de esto en la línea de comandos con el binario Symfony. Empieza ejecutando:

```terminal
symfony project:init
```

Esto arranca unos cuantos archivos platform.sh, que puedes ver aquí.`.platform.app.yaml` contiene instrucciones sobre cómo desplegar - como qué comandos ejecutar, qué versión de PHP utilizar, la configuración del servidor web y más. `services.yaml`
es donde configuras los servicios que necesitas -como bases de datos, colas, etc.- y`routes.yaml` configura tus dominios, y es un poco menos importante. Ah, y también puedes añadir cualquier configuración personalizada de `php.ini` con este archivo.

He ido registrando mis progresos a lo largo del proceso. Así que cuando ejecuto

```terminal
git status
```

sólo muestra estos nuevos archivos. Vamos a confirmarlos y... ¡genial!

## Registrando el proyecto platform.sh

Ahora que tenemos esos archivos locales, tenemos que llamar a la gente de platform.sh y decirles que queremos crear un nuevo proyecto. Lo haremos con

```terminal
symfony cloud:project:create
```

Ya tengo algunos proyectos en Platform.sh...., lo que significa que ya tengo una organización... y ya tiene mi tarjeta de crédito. ¡Ladrones! Si es la primera vez que haces esto, tendrás que seguir unos pasos adicionales. 

Dale un título a tu proyecto, selecciona una región -yo estoy utilizando "eu-5"- y luego te pregunta qué rama será tu entorno de producción. Yo estoy utilizando la rama por defecto `main`.

A continuación, nos pregunta si queremos establecer "Mixed Vinyl" como remoto para este repositorio. Esto es bastante guay porque expone cómo funciona platform.sh. Para desplegar con platform.sh, en realidad empujamos nuestro repositorio git a un repositorio remoto en sus servicios. Ellos lo ven, cogen el código y lo despliegan

De todos modos, voy a decir "no", pero tú puedes decir "sí". Como voy a decir "no", me verás hacerlo manualmente en un minuto - y te explicaré más sobre ello.

Por último, te pide que confirmes el precio. Estos 12 USD al mes son la tarifa de desarrollador que puedes pagar para jugar con las cosas. Será más cara cuando decidas poner tu sitio en producción de verdad. Me encanta platform.sh por lo fácil que me hace la vida, pero hay opciones más baratas.

Tardarás uno o dos minutos en configurarlo todo entre bastidores. Cuando termine... ¡ding! Obtenemos algo de información, incluido el nuevo ID del Proyecto.

Nota al margen: También hay una interfaz web en Platform.sh, así que no todo tiene que hacerse a través de la línea de comandos. Pero, ya sabes, los empollones como yo preferimos la línea de comandos.

## Vincular el código local al proyecto remoto

Copia el ID del Proyecto. Llegados a este punto, tenemos algunos archivos de configuración locales -como`.platform.app.yaml` - y hemos creado un nuevo "proyecto" en plataforma.sh. Pero aún no están vinculados: nuestro código local no sabe que "pertenece" a este proyecto en plataforma.sh.

Para vincularlos, ejecuta

```terminal
symfony project:set-remote
```

y pega el ID del proyecto.

## Nuestro primer despliegue

¡Listo! ¿Listo para desplegar? Hazlo con:

```terminal
symfony deploy
```

Actualmente estamos en la rama "principal", por lo que se está desplegando a, básicamente, nuestra máquina de "producción", lo que se llama un entorno. Una de las partes más interesantes de platform.sh es que desplegarás tu rama `main` en el entorno de "producción", pero también puedes desplegar otras ramas git en otros entornos platform.sh, en los que puedes pensar como otros servidores platform.sh.

De todos modos, como he mencionado, entre bastidores, este comando es sólo un atajo para`git push` nuestra rama a un remoto git en los servidores platform.sh. Y al hacerlo, ¡se inicia el despliegue!

Oooh, esto parece elegante y friki. Aquí hay un montón de detalles, incluida una advertencia sobre el uso de una versión antigua de Composer. Lo arreglaremos en un minuto. Aquí abajo, vemos que se está ejecutando `symfony composer install` y realizando algunos otros pasos: todas las cosas básicas que necesitas para desplegar cualquier aplicación Symfony.

En la parte inferior, nos da un certificado SSL, y si seguimos desplazándonos... oooh, ¡tenemos un mensaje sobre un error en la base de datos! Ignóralo por ahora porque... cuando termina, ¡nos da una URL!

Como no hemos configurado ningún dominio para el sitio, nos da una URL temporal. Cópiala, gira y... ¡nuestro sitio está vivo! No tiene ningún estilo... ya que no hemos hablado de AssetMapper, ¡pero al menos funciona!

¿Pero cómo? ¿Cómo ha sabido ejecutar `composer install` y esas otras cosas? ¿Y la advertencia de Composer y el error de la base de datos? Vamos a profundizar en todo eso a continuación.
