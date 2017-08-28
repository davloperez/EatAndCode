title:
  Dependencias con NPM. Trucos y trampas.
date: 2016-08-13 18:20:00
tags:
- npm
- trucos
- trampas
categories:
- node.js
share_cover: /images/npm-logo.png
---
Node.js utiliza su propia implementación de [Common.js](https://nodejs.org/api/modules.html#modules_modules "Node.js Module Pattern") para el manejo de dependencias y módulos.
Todas las aplicaciones Node.js que hagamos (incluso la más sencilla) van a hacer uso de multitud de módulos para funcionar. 
Estos módulos pueden ser _builtin_, propios del mismo Node.js, o pueden ser módulos de terceros; pero ambos funcionan exactamente igual a la hora de importarlos y usarlos en nuestras aplicaciones.
No es el objetivo de este artículo enseñar el funcionamiento, que podéis encontrar en cualquier guía "_get started_" por la red, sino que iré un paso más allá y comentaré algunos de los trucos y trampas que me he encontrado en mi experiencia de desarrollo con Node.js.

# Los módulos son _singleton_... pero a veces no!
Una de las primeras cosas que aprendemos en el desarrollo con Node.js es que cuando haces un `require('xyz')` por primera vez, el módulo es leído, procesado síncronamente y almacenado en memoria, y si posteriormente hacemos otro `require('xyz')` no se vuelve a procesar, sino que se hace uso del mismo módulo que ya existe en memoria cuando se cargó por primera vez. 
Solemos llamar a este comportamiento como _singleton_, debido a su similitud con este patrón. De hecho, es muy facil implementar objetos _singleton_ en Node.js gracias a Common.js.
Es más, NPM **ni siquiera descargará** el mismo módulo más de 1 vez en el caso de que ya se haya descargado previamente mediante otro módulo diferente que lo requiriese.
Podemos ver este comportamiento en el siguiente ejemplo.

## Ejemplo de uso _singleton_ de un módulo

Supongamos que tenemos una app con el siguiente *package.json*:
```
  ...
  "dependencies": {
    "colors": "^1.1.2",
    "esrol-logger": "0.0.10"
  }
  ...
```
La app tiene 2 dependencias: el módulo `colors` y el módulo `esrol-logger`. 
Este último, a su vez, tiene otras dos dependencias: `colors` y `debug`. 
```
  ...
  "dependencies": {
    "colors": "^1.1.2",
    "debug": "^2.2.0"
  }
  ...
```
Por tanto tenemos el siguiente escenario: tanto nuestra `app` como el módulo `esrol-logger` **requieren** del módulo `colors` para funcionar. O representado en forma de esquema:
```
app
├── colors                                                                                                               
└─┬ esrol-logger   
  ├── colors                                                                                                      
  └── debug
```
Sin embargo, si hacemos `npm install` y luego un `npm list`, veremos lo que se ha instalado realmente.
```
c:\Users\David\Proyectos\eatandcode-npm-dependency-example>npm list                                                              
app@1.0.0 c:\Users\David\Proyectos\eatandcode-npm-dependency-example                               
├── colors@1.1.2                                                                                                                 
└─┬ esrol-logger@0.0.10                                                                                                          
  └─┬ debug@2.2.0                                                                                                                
    └── ms@0.7.1 
```
El módulo `colors` sólo se ha instalado una única vez. 
Cuando el módulo `esrol-logger` quiera hacer uso del módulo `colors`, el manejador de dependencias de Node.js sabrá localizarlo en la carpeta node_modules de nuestra app.
Y lo que es más importante, dicho módulo sólo se cargará 1 única vez, y el resto de veces que se le haga un `require`, se obtendrá la instancia ya cargada en memoria.

Para comprobar que realmente esto sucede así, vamos a llevar a cabo un pequeño _truco_ que nos permitirá ver la cantidad de veces que se procesa el módulo `colors`.
Vamos a la carpeta *node_modules/colors/lib* y editamos el fichero _index.js_. 
Este fichero es el punto de entrada por el que se empieza a procesar este módulo (indicado en el campo "main" de su _package.json_, más información [aquí](https://docs.npmjs.com/files/package.json#main "Documentación oficial") ).
Colocaremos un chivo que nos mostrará por consola un mensaje cuando se procese este fichero. Para ello, añadimos al final del fichero lo siguiente:
```javascript
  console.log('Colors module has been processed');
```
Guardamos el fichero, y ejecutamos nuestra app, que va a hacer uso directamente de los módulos `colors` y `esrol-logger`. Nuestra app consta simplemente del siguiente fichero principal:
```javascript
var colors = require('colors');
var logger = require('esrol-logger');
console.log('App finished');
```
Tras ejecutarla, obtendremos por consola los siguientes mensajes:
```                                                      
Colors module has been processed                                                                                                 
App finished   
```
Como podemos ver, el módulo `colors` sólo se ha procesado una única vez, a pesar de haber sido _required_ tanto por nuestra app como por el módulo `esrol-logger`.
Sin embargo, es posible que, bajo determinadas circunstancias, el módulo `colors` se procese dos veces, como se explica en el siguiente ejemplo.

## Ejemplo de múltiples procesados del mismo módulo.
Ahora supongamos que tenemos la misma aplicación, pero esta vez vamos a "forzar" a nuestra app a utilizar una versión de `colors` diferente a la que utiliza el módulo `esrol-logger`.
`esrol-logger` require una versión del módulo `colors` igual o superior a la 1.1.2. 
Vamos a modificar el package.json de nuestra app para establecer como dependencia la versión exacta 1.0.0.
```
  ...
  "dependencies": {
    "colors": "1.0.0",
    "esrol-logger": "0.0.10"
  }
  ...
```
Borramos la carpeta node_modules por completo, y volvemos a ejecutar `npm install` y luego `npm list`, donde veremos que se ha descargado el siguiente grafo de dependencias.
```
c:\Users\David\Proyectos\eatandcode-npm-dependency-example>npm list                                                              
app@1.0.0 c:\Users\David\Proyectos\eatandcode-npm-dependency-example                                                             
├── colors@1.0.0                                                                                                                 
└─┬ esrol-logger@0.0.10                                                                                                          
  ├── colors@1.1.2                                                                                                               
  └─┬ debug@2.2.0                                                                                                                
    └── ms@0.7.1
```
Si os fijáis bien, el módulo `colors` se ha descargado 2 veces, una para la app (versión 1.0.0) y otra como dependencia de `esrol-logger` (versión 1.1.2). ¿qué implica esto?
Pues bien, ahora al arrancar nuestra app, cabe la posibilidad de que se procese dos veces el módulo `colors`, y de que ambas _versiones_ coexistan en memoria (como si fueran dos singleton independientes). 
Y esto **va a depender de dónde se haga el `require('colors')`**. Si se realiza desde nuestra app, obtendremos la instancia del módulo versión 1.0.0, mientras que si se realizar desde dentro de `esrol-logger`, obtendremos la instancia versión 1.1.2.

Hagamos una prueba similar a la anterior para corroborar esto. 
Modificamos, al igual que hicimos antes, los ficheros _index.js_ que hay en las carpetas *node_modules/colors/lib* de **ambos** módulos `colors`, añadiéndole la líneas siguientes al final de cada cada uno:
```javascript
  console.log('Colors module 1.0.0 has been processed');
```
```javascript
  console.log('Colors module 1.1.2 has been processed');
```

Después simplemente ejecutamos de nuevo nuestra app. Y, sorprendentemente obtendremos lo siguiente por consola:
```
Colors module 1.0.0 has been processed                                                                                           
Colors module 1.1.2 has been processed                                                                                           
App finished    
```
Lo que nos certifica que el mismo módulo `colors` ha sido procesado dos veces, una por cada una de las versiones diferentes que requiere nuestra aplicación en general.

# Conclusión
Aunque en los ejemplos provistos anteriormente este problema de "múltiples singletons del mismo módulo" no sea demasiado grave (ya que no afecta para nada a la ejecución de nuestro programa), existen otras situaciones donde este problema se convierte en un bug realmente difícil de detectar y depurar. Un claro ejemplo de ello es el módulo [mongoose](http://mongoosejs.com/ "Web oficial de mongoose"), que me trajo de cabeza durante unas horas hasta que finalmente dí con la tecla. El escenario era el siguiente:

La app principal tenía una dependencia con una versión de mongoose diferente a la de otro módulo privado que habíamos creado a modo de [DAO](https://es.wikipedia.org/wiki/Data_Access_Object "Data Access Object en Wikipedia") para un modelo concreto de nuestro dominio.
En el app.js inicializábamos la conexión de mongoose con el servidor MongoDB, pero dicha conexión **no aplicaba a la instancia de mongoose que cargaba nuestro módulo DAO** debido al comportamiento que hemos visto anteriormente.
Como consecuencia, al intentar utilizar nuestro DAO, mongoose nos daba un error de que no se había inicializado ninguna conexión.

La verdadera problemática aquí reside en los módulos que contienen datos u objetos "singleton" compartidos por toda la aplicación (como hace mongoose).

Mi consejo: hay que tener mucho cuidado con las versiones de los módulos. Probablemente no sea mala idea hacer un `npm list` de nuestros módulos y ver si se están cargando varias instancias del mismo módulo. 
En principio no debería pasar nada, pero si estos módulos contienen datos singleton compartidos por toda la aplicación, es posible que tengamos un problema grave en un futuro (si no lo tenemos ya).
Y por supuesto, si creáis un módulo propio, tratad siempre que sea posible de no tener datos singleton, sino sólo exponer funciones o constructores aislados.

Podéis descargar el código fuente de los ejemplos [en este repositorio GitHub](https://github.com/davloperez/eatandcode-npm-dependency-example "Repositorio de ejemplo")

