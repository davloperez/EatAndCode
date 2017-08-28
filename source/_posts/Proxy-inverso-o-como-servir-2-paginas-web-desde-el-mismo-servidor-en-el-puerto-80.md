title: >-
  Proxy inverso, o cómo servir 2 páginas web desde el mismo servidor en el
  puerto 80
date: 2016-05-28 19:36:07
tags:
- proxy inverso
- servidor
- Squid
categories:
- sistemas
- proxy
share_cover: /images/squid-basic-reverse-proxy.png
---
Hace unos días me ví en la necesidad de publicar dos sitios web (accesibles en el puerto 80) en un equipo con sólo una IP pública. Normalmente, si los dos sitios web están basados en la misma tecnología y alojados en el mismo servidor web (como IIS o Apache), suele ser suficiente con configurar adecuadamente las opciones que estos programas proveen. Por ejemplo, en [este post](http://www.sherweb.com/blog/how-to-set-up-site-bindings-in-internet-information-services-iis/ "Blog de SherWeb") explica cómo hacerlo para IIS. O también en Apache tenemos los [Virtual Hosts](https://httpd.apache.org/docs/current/vhosts/examples.html "Documentación oficial de apache.org") con los que podemos conseguir el mismo efecto.

Sin embargo, el problema se complica cuando los dos sitios web están implementados en tecnología diferente, como era mi caso. Un sitio web corría sobre Node.js y otro sobre ASP.NET con IIS. Cuando inicias un sitio web con uno de estos programas, ya no puedes iniciar el otro, ya que el puerto 80 ha sido ocupado por el primero. 

Aquí es donde entra en juego un [proxy inverso](https://en.wikipedia.org/wiki/Reverse_proxy "Ver en Wikipedia"), como por ejemplo [Squid](http://www.squid-cache.org/ "Sitio oficial de Squid").

![Esquema de funcionamiento de un proxy inverso](/images/squid-basic-reverse-proxy.png "Esquema de funcionamiento de un proxy inverso")

La idea es la siguiente: el único programa que escuchará en el puerto 80 será nuestro proxy inverso. Así no habrá problemas de compartición de puertos. Por otro lado, configuraremos Node.js e IIS para que escuchen en cualquier puerto libre que haya en el equipo, por ejemplo el 4000 y el 8000 respectivamente. Y por último, configuraremos el proxy inverso para que mapee las peticiones que vayan dirigidas a *sitio1.com* hacia *localhost:4000*, y las peticiones de *sitio2.com* hacia *localhost:8000*.

NOTA: aunque en este ejemplo estamos utilizando el mismo equipo para albergar tanto Squid, como los servidores IIS y Node.js, lo aconsejable sería poder ubicar cada uno de estos procesos en una máquina diferente (siendo el equipo de Squid el único que contiene la IP pública. Los equipos de Node.js e IIS sólo necesitarían estar conectados con el de Squid con IPs privadas). Pero si los dos sitios web no requieren muchos recursos de memoria o CPU, podemos ahorrarnos el coste de esas múltiples máquinas, ubicando todos los procesos en una.

# Cómo configurar Squid en Windows



1. Descargar el binario de Squid para Windows de la [página oficial](http://wiki.squid-cache.org/SquidFaq/BinaryPackages "Binarios de Squid").
2. Instalarlo en nuestro servidor (la instalación es trivial). Tras la instalación, deberíamos ver el icono en el área de notificación de Windows.

   ![Squid se iniciará como un servicio](/images/squid-installed.png "Squid se iniciará como un servicio")

3. Arrancamos los servidores Node.js e IIS, escuchando en las direcciones http://localhost:4000 y http://localhost:8000 respectivamente.
4. Hacemos clic sobre el icono de Squid en el área de notificación, y seleccionamos "Open Squid Configuration". Se nos abrirá el fichero **squid.conf** en el bloc de notas.
5. Borramos todo el contenido del fichero, y en su lugar pegamos la siguiente configuración:
   ```
   # Hacemos que Squid escuche en el puerto 80
   http_port 80 accel
   
   # Configuramos el mapeo de sitio1.com al puerto 127.0.0.1:4000
   cache_peer 127.0.0.1 parent 4000 0 no-query originserver name=nodeJsAccel
   acl sitio1Sites dstdomain sitio1.com
   http_access allow sitio1Sites
   cache_peer_access nodeJsAccel allow sitio1Sites
   cache_peer_access nodeJsAccel deny all
   
   # Configuramos el mapeo de sitio2.com al puerto 127.0.0.1:8000
   cache_peer 127.0.0.1 parent 8000 0 no-query originserver name=iisAccel
   acl sitio2Sites dstdomain sitio2.com
   http_access allow sitio2Sites
   cache_peer_access iisAccel allow sitio2Sites
   cache_peer_access iisAccel deny all
   ```
6. Reiniciamos Squid para que cargue la nueva configuración. Para ello, hacemos clic en el icono del área de notificación de Windows, "Stop Squid service", y después "Start Squid service".

Si lo hemos hecho todo correctamente, ahora podremos acceder a http://sition1.com y Squid nos redirigirá de manera totalmente transparente hacia el servidor Node.js que escucha en el puerto 4000. Lo mismo con http://sition2.com, que nos redirigirá hacia el servidor IIS que escucha el puerto 8000. Todo esto sucede de manera interna en nuestro servidor, y el usuario final no notará diferencia algura respecto a si tuviésemos los sitios web corriendo en equipos diferentes. Obviamente, en el paso **5** deberemos sustituir *sition1.com* y *sition2.com* por nuestros dominios reales, así como los puertos privados en los que tengamos corriendo nuestras aplicaciones web. Además, podemos añadir tantos mapeos a Squid como queramos, permitiéndonos tener multitud de sitios web en un mismo equipo, todos ellos accesibles desde el puerto 80.

Como apunte final, decir que Squid sirve para muchas otras cosas además de un simple proxy inverso. Yo aún estoy verde con este programa, pero espero aprender más sobre él y mostrar algunos de sus prácticos usos.

