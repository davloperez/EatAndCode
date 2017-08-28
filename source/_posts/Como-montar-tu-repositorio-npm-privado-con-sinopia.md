title: Cómo montar tu repositorio NPM privado con Sinopia
tags:
  - Sinopia
  - npm
  - repositorio
categories:
  - Node.js
share_cover: /images/sinopia-web.png
date: 2016-06-07 21:48:18
---
Esta tarde, un amigo me ha preguntado si conocía algún símil de "[Nexus](http://www.sonatype.org/nexus/ "Página oficial de Nexus")" para **node.js**, con el que poder gestionar las versiones de sus librerías y componentes privados. Me resulta curioso, ya que hace unas pocas semanas yo también estuve investigando al respecto. Aunque no es exactamente un nexus, el propio gestor de paquetes NPM sirve para precisamente eso, pero el problema es que si publicas un componente al repositorio NPM central, éste será públicamente accesible por todo el mundo. Por supuesto, siempre puedes contratar la versión [Enterprise de NPM](https://www.npmjs.com/enterprise "Versión Enterprise de NPM") y olvidarte de todo lo que viene a continuación. Pero si no te sobra la pasta, probablemente prefieras seguir leyendo para ver qué soluciones alternativas gratuitas hay.

Una alternativa podría ser aprovechar la capacidad de NPM de funcionar con repositorios de GitHub. La idea sería crear un repositorio privado en GitHub y almacenar ahí nuestros componentes privados que usaremos en otras partes de nuestra aplicación. De hecho, no es necesario disponer de un repositorio privado para cada componente, como se explica [en esta página](http://www.zerothedragon.com/737/repositorio-con-multiples-paquetes-de-node/ "Zero the dragon"). Aunque puede servirnos para salir del paso, personalmente me gusta mucho más otra alternativa que es más sencilla y transparente a la hora de trabajar: [Sinopia](https://github.com/rlidwka/sinopia "Ver Sinopia en GitHub").

NOTA: he leído que existen otras alternativas como [Reggie](https://github.com/mbrevoort/node-reggie "Ver Reggie en GitHub") o [Kappa](https://github.com/krakenjs/kappa "Ver Kappa en GitHub"), pero personalmente sólo he llegado a probar Sinopia.

## Sinopia, un proxy entre NPM Central y tu entorno de desarrollo.

Si trabajas en un equipo de varios programadores, probablemente quieras disponer de un equipo central en el que poder ubicar las versiones de los componentes y librerías Node.js que vayáis creando. Desde ese equipo central, los programadores nos descargaremos tanto los paquetes públicos NPM como los nuestros privados. Sinopia hará las dotes de "proxy" entre NPM Central y nuestros equipos de desarrollo, cogiendo los paquetes NPM de su memoria local o descargándoselos de NPM Central en caso de no existir. Nótese que además esto nos puede dar la ventaja de acelerar los tiempos de descarga y prevenir los problemas que nos podría ocasionar una caída temporal de NPM Central (aunque seamos realistas, su velocidad de descarga y su disponibilidad son realmente buenos y rara vez tendremos problemas).

![Diagrama de un proxy Sinopia](/images/sinopia.png "Diagrama de un proxy Sinopia")

En el esquema anterior podemos ver donde se ubicaría Sinopia. Cabe destacar que, para hacer pruebas lo podemos instalar en nuestra propia máquina de desarrollo (de hecho, así lo he hecho yo en el ejemplo que podré a continuación).

## Cómo configurar Sinopia de manera sencilla en Windows

En primer lugar, abrimos una consola e instalamos Sinopia de manera global:
```
npm install -g sinopia
```
No os preocupéis si veis algunos errores en la consola, es normal.
A continuación, sencillamente lanzamos sinopia con el siguiente comando:
```
sinopia
```

![Resultado por consola al lanzar Sinopia](/images/sinopia-cmd.png "Resultado por consola al lanzar Sinopia")

Al ser la primera vez que lanzamos, nos creará automáticamente un fichero de configuración bajo nuestra carpeta de usuario */AppData/Roaming/sinopia/config.yaml*. También se creará una carpeta vecina llamada *storage*, que será donde Sinopia almacenará los paquetes NPM públicos y privados. Podéis modificar el archivo de configuración para configurar a vuestro antojo el comportamiento de Sinopia. Para una guía completa y detallada, consultad la documentación oficial en GitHub.

Con esto, **ya tendremos listo el servidor de Sinopia** para hacer de proxy con los paquetes NPM públicos y para almacenar nuestras librerías privadas. De hecho, podemos incluso navegar por una interfaz web local bastante chula, muy similar a la de NPM Central, donde podremos consultar nuestros paquetes privados. Dicha interfaz se encuentra en [http://localhost:4873/](http://localhost:4873/).

![Interfaz web de Sinopia](/images/sinopia-web.png "Interfaz web de Sinopia")

Pero nos falta una última cosa por configurar; necesitamos decirle a NPM que debe obtener y publicar los paquetes de nuestro proxy Sinopia en vez de NPM Central. Para ello, abrimos otra consola y ejecutamos el siguiente comando:
```
npm set registry http://localhost:4873
```
Esto establecerá esa configuración en el NPM de nuestra máquina. Obviamente, en caso de estar Sinopia ubicada en otro equipo, tendremos que indicar ahí la IP de dicho equipo en lugar de *localhost*. Además, tendremos que configurar un usuario npm que pueda realizar "commits" de los paquetes privados a nuestro proxy. Ejecutamos el siguiente comando en la nueva consola que hemos abierto antes:
```
npm adduser --registry http://localhost:4873
```
![Configuración del usuario en el registro de NPM](/images/sinopia-user.png "Configuración del usuario en el registro de NPM")

## Ejemplo de uso

Vamos a crear una aplicación de pruebas para ver que funciona correctamente. Creamos una carpeta llamada "test" en cualquier lugar de nuestro equipo, y en su interior ejecutamos el siguiente comando:
```
npm init
```
Tras la serie de preguntas típicas, se nos habrá creado el fichero project.json, con los datos que hemos introducido antes en su interior. Para este ejemplo, yo he establecido la siguiente configuración:
```json
{
  "name": "eat-and-code",
  "version": "1.0.0",
  "description": "A simple testing package",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```
Ahora vamos a añadirle una dependencia de un paquete público de NPM, por ejemplo, el súper útil paquete [status-cat-bot](https://www.npmjs.com/package/status-cat-bot "Ver status-cat-bot"). En esta misma carpeta, ejecutamos lo siguiente:
```
npm install http-status-cats --save
```
Si lo hemos hecho todo correctamente, en el interior de la carpeta de Sinopia/storage debería haberse creado una carpeta con la librería http-status-cats. Recordemos que esta librería es descargada por Sinopia desde el repositorio central de NPM.

![Carpetas descargadas por la caché de Sinopia](/images/sinopia-cache.png "Visualización de las carpetas descargadas por la caché de Sinopia")

Ahora vamos a probar la subida de nuestra librería "test" que hemos creado. Sencillamente tenemos que ejecutar el siguiente comando:
```
npm publish
```
Nuestro paquete NPM privado de test se habrá publicado sólo a nuestro proxy Sinopia, y no a NPM Central, y sólo aquellos compañeros de trabajo que tengan acceso a ese proxy podrá utilizar esta librería. Podemos consultarlo desde la interfaz web:

![Ejemplo de nuestra librería publicada sólo a nuestro repositorio privado](/images/sinopia-web-example.png "Ejemplo de nuestra librería publicada sólo a nuestro repositorio privado")

**NOTA IMPORTANTE**

Si queremos volver a dejar la configuración de NPM para que haga uso del repositorio NPM Central, ejecutamos esto en una ventana de comandos:
```
npm config set registry https://registry.npmjs.org
```