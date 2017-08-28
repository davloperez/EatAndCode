title: TypeScript 2, control del flujo basado en análisis de tipos
date: 2016-06-04 10:45:57
tags:
- Typescript 2
- Javascript
categories:
- Typescript
share_cover: /images/typescript.png
---
Ayer estaba leyendo el blog de [Juan María Hernández](http://blog.koalite.com/ "Koalite") cuando me encontré con un enlace muy interesante en su [revisión personal del lenguaje TypeScript](http://blog.koalite.com/2016/05/typescript-ahora-si/ "Typescript ¿Ahora sí?"). Se trata de un vídeo de [Channel 9](https://channel9.msdn.com/ "Channel 9 de Microsoft") en el que aparece Anders Hejlsberg comentando algunas novedades que traerá la próxima versión de TypeScript. Os aconsejo que veáis el vídeo, es muy interesante.

<iframe src="https://channel9.msdn.com/Blogs/Seth-Juarez/Anders-Hejlsberg-on-TypeScript-2/player" width="100%" height="424" allowFullScreen frameBorder="0"></iframe>

Hejlsberg habla primero de TypeScript en general, y luego realiza un avance sobre algunas mejoras que traerá la versión 2 de este lenguaje. De estas mejoras, explica que traerá async y await compilable a ECMAScript 5, y la parte que más me llamó la atención, los tipos *nullables* con **control del flujo basado en análisis de tipos**.

## Una ayuda para evitar los errores por referencias nulas

El concepto *control del flujo basado en análisis de tipos* suena a algo muy complicado de entender, pero en realidad es algo que muchos de nosotros hemos deseado siempre tener en nuestro IDE como programadores. En palabras sencillas, se trata de que el compilador nos indique mientras escribimos el código de manera (casi) instantánea que estamos introduciendo un posible error de tipo *"Cannot read property 'xxxx' of null"* (o undefined). Aunque ya existían en el mercado algunos *[Analyzers](https://en.wikipedia.org/wiki/List_of_tools_for_static_code_analysis)* de código que podíamos usar de manera externa o integrada en nuestro IDE, TypeScript 2 traerá esta funcionalidad consigo de manera "nativa", lo cual es realmente sorprendente; pensad, que estaremos desarrollando en realidad con un lenguaje no tipado como es Javascript, en el que no sólamente se le han añadido "tipos" en una capa superior con TypeScript 1, sino que ahora seremos capaces de prevenir y anticiparnos a bugs y errores debido a la nulabilidad de esos tipos.

Esto se entiende mucho mejor con un ejemplo sencillo. Usaré el mismo que propone en el vídeo superior.

```javascript
function foo(x: number|null){
	x.toString();
}
```

En este ejemplo, el nuevo compilador de TypeScript 2 nos indicaría una advertencia de manera inmediata por el hecho de estar llamando al método "toString()" de una variable que podría ser null en tiempo de ejecución. Y lleva toda la razón. Sin embargo, si optamos por escribir nuestro código de esta manera:

```javascript
function foo(x: number|null){
    if(x !== null){
        // aquí sabemos que x jamás podrá ser null
        x.toString();
    }
    else{
        // y aquí sabemos con certeza que x será en todo caso null
    }
}
```
En este caso, la llamada a x.toString() nunca se puede llegar a ejecutar si x es null. Gracias al *control del flujo basado en análisis de tipos*, el compilador será lo suficientemente inteligente como para detectar estos casos, y no mostrará ninguna advertencia. ¡BUUMM!

Este ha sido un ejemplo muy sencillo, pero lo que realmente me inquieta a mi es saber si de verdad va a funcionar en situaciones más complejas que se pueden dar en el código, como por ejemplo esta:
```javascript
function foo(x: number|null){
	return bar(x, (typeof x === 'number'));
}
function bar(x: number|null, y: boolean){
    while(y){
        // en este bucle sólo entrará si x es un número y no es null
        x.toString();
        if(x > 10){
            x = x - 1;
        }else{
            x = null;
            y = false;
        }
    }
}
```

Si a eso le sumamos algoritmos recursivos con varias funciones... la lógica se complica, y mucho. Lo que no llego a comprender del todo es cómo harán para que este mecanismo funcione sin tener que probar y, de hecho, ejecutar el algoritmo con todos los valores posibles de entrada, ya que eso creo que es inviable. Supongo que la solución estará relacionada con cómo funciona el compilador, pero me sorprende muchísimo hasta qué punto se puede llegar a inferir el comporamiento de una porción de código aleatorio.

Habrá que ver como funciona (dicen que en los [*nightly builds*](https://github.com/Microsoft/TypeScript/wiki/Nightly-drops "Nightly drops en GitHub") ya se puede probar esta funcionalidad), pero desde luego es una ayuda que me encantaría tener en mi trabajo diario.