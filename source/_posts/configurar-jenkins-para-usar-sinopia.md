title: Cómo hacer que Jenkins utilice una configuración de NPM específica
tags:
  - Sinopia
  - npm
  - repositorio
  - Jenkins
  - test
categories:
  - Node.js
  - Integración continua
share_cover: /images/jenkins-npm.png
date: 2016-08-20 14:15:00
---
[Jenkins](https://jenkins.io/ "Página web oficial de Jenkins") es uno de los entornos de integración continua más conocidos, sobre todo en el entorno Java, pero que gracias a su sistema de plugins y a su versatilidad, también puede ser utilizado en proyectos de muchas otras tecnologías.

Una de las posibilidades que nos ofrece es la integración con Node.js, de manera que podemos ejecutar, por ejemplo, los test Mocha de nuestros proyectos, y obtener una visualización de los resultados e históricos (además de la querida funcionalidad del envío de emails cuando alguien hace cambios en el código que provoquen el fallo de algún test). Si queréis saber como configurar Jenkins para esto, podéis verlo en este [estupendo artículo](https://blog.dylants.com/2013/06/21/jenkins-and-node/ "CONTINUOUS INTEGRATION WITH JENKINS AND NODE.JS")

## El problema de NPM con Jenkins en Windows
**NOTA: desconozco si este problema también ocurre en Linux o Mac. Si alguno de vosotros quiere probarlo, puede poner los resultados en los comentarios.**

Si seguimos el artículo que he puesto en el párrafo anterior, no tendremos ningún problema para poner en marcha nuestro entorno de CI para la ejecución de nuestros test unitarios, **excepto si utilizamos algún tipo de configuración especial de NPM**. Cuando digo "configuración especial" me refiero a, por ejemplo, si haces uso de [Sinopia como repositorio NPM privado](http://eatandcode.es/2016/06/07/Como-montar-tu-repositorio-npm-privado-con-sinopia/ "Cómo montar tu repositorio NPM privado con Sinopia"), o si es necesaria la autenticación para la descarga de algún paquete NPM.

Esta configuración de autenticación, URL del registro, etc. habitualmente se almacena en el fichero [.npmrc](https://docs.npmjs.com/files/npmrc "Especificación oficial del fichero .npmrc") que reside en nuestra carpeta Home de usuario (en Windows suele estar en _C:\Users\David_)

Pero el problema está en que Jenkins no utiliza las mismas variables de entorno que disponemos en una consola CMD común, y por tanto, cuando ejecutamos algún comando NPM, éste no es capaz de acceder a la configuración de autenticación o de registro que tenemos en nuestro fichero .npmrc.

Este es un ejemplo del log que muestra Jenkins cuando tratamos de ejecutar el comando `npm install` desde un Job y en el _package.json_ tenemos como dependencias algunos de nuestros módulos privados en Sinopia.

![Log del error al ejecutar npm install](/images/jenkins-npm-error-404.png "Log del error al ejecutar npm install")

En la imagen anterior podemos ver cómo al hacer `npm install` NPM trata de obtener el paquete privado **xs-wf-event-type-service** del repositorio central de NPM, en vez de buscarlo en nuestro servidor Sinopia.

## La solución está en las variables de entorno
Existen algunos [plugins de Jenkins](https://wiki.jenkins-ci.org/display/JENKINS/NodeJS+Plugin "NodeJS Plugin") que parece ser que nos dejan configurar diferentes instalaciones y configuraciones de Node.js y NPM. El inconveniente es que por el momento sólo está disponible para Linux. Si tu caso, como el mío, es que estás en un entorno Windows, podrás resolver este problema de la manera que explico a continuación.

La solución se basa en saber que NPM utiliza la variable de entorno `USERPROFILE` para determinar dónde buscar el fichero .npmrc. Y esa variable de entorno no está definida cuando ejecutamos un script CMD desde Jenkins. Por tanto, simplemente tenemos que definirla en la configuración de Jenkins y darle el valor adecuado para que localice nuestra carpeta Home.

Accedemos a esta sección desde Configurar Jenkins -> Configurar el Sistema -> Propiedades Globales

![Configuración de las variables de entorno en Jenkins](/images/jenkins-environment-variables.png "Configuración de las variables de entorno en Jenkins")

Guardamos, y ahora sí podremos ejecutar `npm install` desde nuestros Jobs con la certeza de que van a utilizar nuestra configuración personalizada que tengamos almacenada en el fichero .npmrc.
