---
title: "Exportar tabla con JQuery"
date: 2012-11-12
tags: ["web"]
---

¿Quién nunca ha tenido la necesidad de exportar un reporte tabulado a MS Excel?

![_config.yml](/images/exportar-tabla-con-jquery.png)

En la red podemos encontrar muchas alternativas para lograr éste objetivo: librerías para PHP, módulos para .Net , etc. 

Sin embargo, una solución para cualquier plataforma de desarrollo Web viene de la mano de jQuery. Gracias a un plugin que permite copiar el contenido de una tabla a partir de su ID.

Para acceder a éste plugin, podemos descargar los archivos necesarios desde [aqui](http://kayalshri.github.io/tableExport.jquery.plugin/).

Si bien en [ése](http://kayalshri.github.io/tableExport.jquery.plugin/) sitio hay opciones para poder exportar una tabla a varios formatos (PDF, CSV, PNG, etc.), lo que se va a explicar aquí es la forma de exportar una Tabla a Excel:

Una vez tengamos los archivos y se hayan subido a nuestro respectivo servidor web, debemos referenciarlos en el header de la página donde se tiene la tabla para exportar:

~~~html
<script type="text/javascript" src="tableExport.js">
<script type="text/javascript" src="jquery.base64.js">
~~~

Luego, en alguna parte de nuestra página, creamos un Boton para exportar nuestra tabla:

~~~html
<button class="btn btn-mini download-image" title="Descargar Excel" 
onClick="$('#id_tabla').tableExport({type:'excel',escape:'false'});">
    <img src="'.base_url().'img/excel.png" />
</button>
~~~

Recordar que nuestra tabla de tener el id "id_tabla":

~~~html
<table id="id_tabla" class="table table-striped">
...
</table>
~~~

Con éso ya tenemos nuestro exportador de Tablas a Excel