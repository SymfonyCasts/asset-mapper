# Desplegar los activos

¿Cómo introducimos nuestros activos en el sitio? Si "ves la fuente de la página", parece que las cosas funcionan. Vemos el `importmap` y... aquí abajo, estas rutas parecen correctas: incluso tienen la parte de la versión en sus nombres de archivo.

## Compilar activos para producción

Por desgracia, todos estos archivos devuelven un 404. Boo. En el entorno `dev`, cuando trabajamos localmente, estos archivos no existen físicamente. Pero un oyente interno de Symfony intercepta la petición, encuentra el archivo y los sirve.

Pero en el entorno `prod`, ese sistema ni siquiera está activo. Es demasiado lento para ejecutarse en producción... así que todo se queda en 404. ¡Y no pasa nada! Hace mucho tiempo, conocimos el comando para solucionar esto:

```terminal
php bin/console asset-map:compile
```

El trabajo de este comando es sencillo: coge todos los activos que AssetMapper conoce y los mueve al directorio `public/assets/`. No es un comando que necesites ejecutar localmente, pero es algo que necesitas ejecutar cuando despliegues.

Copia esto, dirígete a `.platform.app.yaml`, y baja al paso `build`. ¡Esto está muy bien! Vamos a dejar que Symfony haga lo suyo en `build`, y después, añadiremos nuestras propias cosas. Justo aquí, añade `php bin/console asset-map:compile`. ¡Ya está!

[[[ code('5b64b210d7') ]]]

¿Por qué ejecutamos esto durante `build` y no durante `deploy`? Como regla general, si el trabajo de un comando es "preparar" archivos, debería estar en el paso `build`. O, otra forma de pensar en ello es: si un comando no requiere una conexión a la base de datos ni a ningún otro servicio en ejecución, hay muchas probabilidades de que sea algo de "construcción".

***TIP
Mantén tu paso "desplegar" lo más rápido posible porque las peticiones entrantes se retienen hasta que finaliza. Puedes encontrar más información aquí: https://docs.platform.sh/overview/build-deploy.html#deploy-steps
***

Vuelve aquí y ejecuta:

```terminal
git add -p
```

Ese espacio en blanco me molesta... así que lo arreglaré y preservaré mi cordura. Luego ejecuta`git commit -m` con un mensaje elegante.

```terminal-silent
git commit -m "asset-map:compile"
```

Ya sabes lo que sigue. ¡Pégale un puñetazo!

```terminal
symfony deploy
```

Adelantémonos a la parte buena. ¡Ya está aquí! ¡Lo vemos ejecutando el comando!

> Compilando activos a `/app/public/assets/`
> Compila 16 activos

También escribe algunos otros archivos dentro del directorio `public/assets/`:`manifest.json` y `importmap.json`. Ayudan a Symfony a volcar el `importmap` y otras cosas en la página aún más rápido.

Y... ¡listo! Vuelve a girar, actualiza y... ¿¡todavía tiene mal aspecto!? Ah, ¡pero las cosas no están tan mal como crees! Ve a la página principal y abre tu Consola. ¡Ejecuta nuestro JavaScript! ¡Vemos el `console.log()`!

## Construyendo Tailwind en Deploy

Así que JavaScript, ¡comprobado! CSS... no tanto. Seguimos teniendo un 404 en`app.tailwind.css`.

Recuerda: cuando veas un 404 a un nombre de archivo que no incluye una parte de versión, como aquí, significa que AssetMapper no puede encontrar ese archivo. ¿Puedes detectar el problema? Este `app.tailwind.css` es un archivo que estamos construyendo... ¡y no está confirmado en el repositorio! Detendré este comando y lo volveré a ejecutar para que podamos ver los detalles. Sí, estamos construyendo el archivo `app.tailwind.css`, ignorándolo de Git, y como Platform.sh se despliega utilizando nuestros archivos de Git, ese archivo simplemente falta.

No pasa nada. Esto es sólo otra cosa que tenemos que añadir a nuestro paso `build`... antes de ejecutar `asset-map:compile` para que el archivo esté disponible.

Voy a pegar el código correspondiente. Se trata básicamente del mismo código que ejecutamos antes para configurar las cosas localmente, salvo que estamos utilizando la compilación de `linux-x64`. Lo descargamos, lo movemos al directorio `/bin` (no importa dónde vaya), lo hacemos ejecutable y ejecutamos el mismo comando para que el archivo de salida esté disponible cuando se ejecute`asset-map:compile`.

[[[ code('5faf3b125e') ]]]

Ah, y no te olvides del TailwindBundle, que facilita un poco el uso de Tailwind, incluido este paso de despliegue.

Volvamos aquí, confirmemos el nuevo cambio de configuración .... y despleguemos de nuevo:

```terminal-silent
symfony deploy
```

Incluso mientras se despliega, podemos ver que esto funciona. La última vez había 16 archivos, ahora hay 17. Cuando termine, gira, actualiza y... ¡está vivo! Todas las páginas tienen CSS. ¡Me encanta!

Ahora que estamos en producción, hablemos de las cosas que tenemos que comprobar para asegurarnos de que nuestros activos se sirven rapidísimamente.
