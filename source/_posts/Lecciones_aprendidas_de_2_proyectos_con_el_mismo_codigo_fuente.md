---
title: Lecciones aprendidas de 2 proyectos con el mismo código fuente
date: 2016-06-02 1:39:14
tags:
- MEAN Stack
- OAuth 2.0
- Framework
categories:
- web
- arquitectura
share_cover: /images/wonder-woman.jpg
---
Hace un tiempo, un cliente nos pidió a mí y a mi equipo que le desarrolláramos un prototipo para una herramienta de creación de test psicotécnicos. Disponíamos de tiempo de sobra para hacerlo, de modo que aprovechamos para investigar sobre tecnologías diferentes a las que habíamos usado hasta ahora. Nos decantamos por hacerlo todo con [MEAN Stack](http://www.cantabriatic.com/arquitecturas-web-y-mean-stack/) y orientando la arquitectura hacia microservicios (sencilla, todo muy manual y sin complicarnos demasiado, que era para un prototipo).

Al poco tiempo de comenzar el desarrollo, cuando aún no llebábamos ni el 40% del trabajo hecho, el mismo cliente nos pidió hacer otro nuevo prototipo para una cosa totalmente diferente, una herramienta de análisis de voz. Enseguida nos dimos cuenta de que los dos proyectos tenían una serie de características comunes:
- Los dos requerían de autenticación con usuario y password.
- Los dos debían tener una interfaz de administración y otra para el resto de usuarios (es decir, necesitábamos crear un sistema de permisos o categorías de usuarios).
- Ambos debían ser multiidioma.
- A nivel de maquetación frontend, ambos eran similares en cuanto a la estructura principal de los elementos en pantalla.
- Los dos proyectos serían MEAN Stack.
- Las "compilaciones" de ambos proyectos serían similares: minificación y empaqueado de recursos, compilaciones específicas para entornos de pruebas y de producción, etc.

Con todos estos puntos en común, parecía obvio que los proyectos nos estaban diciendo a gritos que reutilizásemos componentes comunes a ambos. Pero tras mucho meditarlo, decidimos ir un poco más allá y directamente **implementar ambos proyectos bajo el mismo código fuente**.

## ¿Cómo que "dos proyectos con el mismo código fuente"?
El código fuente era el mismo para ambos, el mismo repositorio de código, etc. En realidad, ese "*framework*" de 2 proyectos era en sí un único proyecto llamado **wonder**. Wonder porque era maravilloso, o al menos eso pensábamos al principio. Un código fuente que nos permitía crear 2 y más aplicaciones sin tener que ver nada la una con las otras... suena maravilloso, mágico, ¿verdad? wonder.

Aunque parezca mentira, para la implementación del 80% de los dos prototipos, wonder nos ahorró muchísimo trabajo. El sistema de autenticación era el mismo para ambas (implementamos un API [OAuth 2.0](http://oauth.net/2/) únicamente con la funcionalidad que necesitábamos), por lo que si encontrábamos un bug en este sistema, al solucionarlo se solucionaba en **ambos proyectos a la vez**. Lo mismo ocurría con la i18n, con el sistema de permisos, etc.

![La 'wonder woman' que querrás ver en el cine, no en tu código fuente](/images/wonder-woman.jpg "Wonder woman")

El problema vino con el 20% restante. Son esa serie de detalles que, aunque fuesen muy sutiles, diferenciaban en algunos aspectos a ambas aplicaciones. Por ejemplo, la aplicación de los test psicotécnicos requería que los usuarios se pudiesen asignar a "grupos", entre los cuales compartían una serie de información relacionada a los tests que iban pasando; sin embargo, en la aplicación de análisis de voz estos grupos no existían, ni siquiera un concepto similar (cada usuario era totalmente independiente del resto). Cuando te encuentras una situación así, te ves en la necesida de decir: "Vale, pues entonces el *framework* wonder deberá exponer un parámetro de configuración llamado *'gruposPermitidos'*, que podrá ser *true* o *false*. Entonces con Gulp, cuando estemos compilando la aplicación, se generará el código fuente final con esa característica o sin ella, según el valor de esa variable". O también otro ejemplo: una de las aplicaciones debía estar en español e inglés, y la otra en español, inglés y portugués. ¿Cómo se resuelve esto? Pues poniendo X más **variables de configuración** que permitan indicar los idiomas disponibles, el idioma por defecto, etc.

Si empiezas a tirar del hilo, te das cuenta de que para poder conseguir esas "diferencias sutiles" entre ambos proyectos, probablemente tendrás que hacerlo de una manera mucho más compleja para que sea un mecanismo genérico, desacoplado de aplicaciónes concretas, cuando en realidad el modo de hacerlo si el proyecto fuese destinado para una única aplicación sería *mucho más sencillo y directo*.

## ¿Hasta qué punto merece la pena compartir código fuente entre proyectos?
Compartir el código fuente mediante un *framework* tipo wonder podría merecer la pena si se dan los siguientes puntos:
- Si sabes que va a servir para 3 o más aplicaciones. Si es sólo para 2 aplicaciones, probablemente no merezca la pena.
- Si las aplicaciones son *prototipos* sencillos de aplicaciones, que sirvan para testear una hipótesis o comprobar la atracción del mercado.
- Si, obviamente, las diferencias entre las aplicaciones son muy (muy muy) pocas.


Que se den estos 3 puntos es bastante poco probable, por lo que como regla general mi consejo es que si os encontráis en una situación similar a la mía, no optéis por compartir el código fuente. En su lugar, intentad buscar una alternativa basada en la paquetización (npm, NuGet..) de las partes comunes de ambas aplicaciones, y **mantened el código fuente de cada aplicación por separado**, añadiendo vuestras dependencias privadas necesarias (vía npm o del modo que sea) para reutilizar la funcionalidad entre ambas aplicaciones.

Esto último es *fácil de decir, pero difícil de hacer*, lo sé :) En un futuro explicaré cómo se puede implementar una solución como esa sin llegar a perder la cabeza en el intento.


