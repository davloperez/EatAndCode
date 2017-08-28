title: Soluciones a la captura de SVG con html2canvas
tags:
  - javascript
  - html2canvas
  - svg
  - jsplumb
  - angularjs
  - material
  - design
  - icon
categories:
  - Javascript
  - Material Design Icons
  - JsPlumb
share_cover: /images/html2canvas-portada.jpg
date: 2017-08-12 23:48:25
---

Hace poco me he visto en la necesidad de tomar capturas de pantalla de una aplicación web mediante javascript, y almacenar dichas capturas como imágenes JPG de manera automática en el servidor.

![html2canvas, imagen de la web del proyecto](/images/html2canvas-portada.jpg)

No hay muchas opciones donde elegir actualmente si quieres poder ejecutar el código que captura la imagen en el cliente, y menos que sean de uso gratuito. La opción más conocida es [html2canvas](https://html2canvas.hertzen.com/ "html2canvas - Screenshots with javascript"). Esta librería es realmente facil de usar; simplemente le proporcionas un elemento del DOM, y la librería te devuelve una imagen en base64. En realidad, suele funcionar bastante bien, pero cuando el nodo del DOM a capturar contiene elementos SVG, existen varios problemas para hacerlo funcionar correctamente. En concreto, los problemas los tuve con los elementos SVG de [Material Design Icons](https://materialdesignicons.com/ "Material Design Icons") usados con el atributo 'md-svg-icon', pero sobre todo con las flechas que genera la librería [JsPlumb](https://jsplumbtoolkit.com/ "jsPlumb Toolkit - build Flowcharts, Diagrams and connectivity based applications fast").

Si html2canvas no captura correctamente vuestros elementos SVG, probad con el siguiente método.

1. (opcional) Si es posible, cread un clon del elemento del DOM cuya imagen queréis capturar. Para ello os podéis servir de la función [.clone()](https://api.jquery.com/clone/ "jQuery API") de jQuery o similares.
2. (opcional) Si habéis creado un clon como he dicho en el punto anterior, deberéis añadirlo al DOM en algún punto donde esté visible (es decir, que no esté bajo ningún nodo con _display: none_). En mi caso, lo pude poner como primer elemento hijo del &lt;body&gt;, ya que el resto de la web estaba alojada en otro &lt;div&gt; que ocupaba el 100% del ancho y del alto, y tenía un z-index superior.
3. Debéis aseguraros de que la etiqueta &lt;svg&gt; y todas las etiquetas interiores como &lt;path&gt; tengan __exactamente__ el atributo `xmlns="http://www.w3.org/2000/svg"`. En mi caso, estaba usando una versión antigua de JsPlumb que creaba las flechas con un elemento SVG que tenía el atributo `xmlns="http://www.w3.org/1999/xhtml"`, el cual no parece funcionar bien con html2canvas.
4. La anchura y altura del SVG deben estar especificado en píxeles en los atributos `width` y `height` del elemento (no mediante CSS ni el atributo `style`). Si utilizáis porcentajes para la anchura y altura, debéis tener en cuenta que se comportará como cualquier elemento con `display:block`. Este es uno de los problemas que ocurren si utilizáis los iconos de Material Design Icons mediante un SVG embebido con el atributo `md-svg-icon` (estos SVG se crean con las propiedades `width="100%"` y `height="100%"`).
5. El posicionamiento de los SVG no puede ser absoluto. En el caso de que tengáis un elemento SVG con `position: absolute`, ésto no se pintará correctamente con html2canvas. Para solucionarlo, podéis envolver los elementos SVG con un &lt;span&gt; que sí tenga posicionamiento absoluto (copiando el posicionamiento top, bottom, left o right del SVG), y borrando del SVG su `position: absolute` con los correspondientes estilos top, bottom, left o right que tuviese. Para envolver los SVG, podéis ayudaros de la función [.wrap()](https://api.jquery.com/wrap/ 'jQuery API') de jQuery.
6. La técnica de `fill:currentColor` para [mostrar el icono SVG con el mismo color que la fuente](https://css-tricks.com/cascading-svg-fill-color/ "Cascading SVG Fill Color") tampoco funciona con html2canvas. Deberéis asignar _progamaticalmente_ el atributo `fill` del SVG con un color HTML real.

A continuación os pongo un fragmento simplificado de lo que he tenido que hacer para obtener unas capturas 100% iguales al HTML original, cuando éste tenía elementos SVG creados por JsPlumb e iconos de angularjs de Material Design Icons. Suponemos que `$clone` es el objecto jQuery que contiene el clon del HTML cuya imagen queremos obtener.

``` javascript
$clone.find('.jtk-connector').each(function () {
  // for every SVG element created by JsPlumb for connections...
  var left = parseInt(this.style.left, 10) + 'px';
  var top = parseInt(this.style.top, 10) + 'px';
  this.removeAttribute('style');
  this.removeAttribute('position');
  this.setAttribute('width', parseInt(this.getAttribute('width'), 10)  + 'px');
  this.setAttribute('height', parseInt(this.getAttribute('height'), 10) + 'px');
  this.setAttribute('preserveAspectRatio', 'xMidYMid meet');
  this.setAttribute('xmlns', 'http://www.w3.org/2000/svg');
  // this.children[0] is the path for connection line
  // this.children[1] is the path for connection arrow shape
  this.children[0].setAttribute('xmlns', 'http://www.w3.org/2000/svg'); 
  this.children[1].setAttribute('xmlns', 'http://www.w3.org/2000/svg');
  this.setAttribute('viewbox', '0 0 ' + parseInt(this.getAttribute('width'), 10) + ' ' + parseInt(this.getAttribute('height'), 10));
  this.children[0].setAttribute('stroke-width', '2px');
  this.children[0].setAttribute('stroke', '#c9c9c9');
  this.children[1].setAttribute('fill', '#c9c9c9');
  this.children[1].setAttribute('stroke', '#c9c9c9');
  $clone.find(this).wrap('<span style="position: absolute; left: ' + left + '; top: ' + top + ';"></span>');
});
```

Seguramente hay alguna forma mejor y más eficiente para solventar este problema, pero este workaround puedo asegurar que funciona correctamente.

Si conocéis algún truco más relacionado con html2canvas, no dudéis en compartirlo en los comentarios.