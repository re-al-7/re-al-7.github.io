---
title: "Npm, Bower y Gulp en ASP NetCore"
date: 2017-06-23
tags: ["desarrollo"]
---

Hola! El dia de ayer, 16 de Agosto, se liberó oficialmente la versión 2.0 de NetCore. Asi que por fin me animé a probarla.

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-01.png)

Ya llevo avanzadas varias cosas en este nuevo mundo, pero ahora hablaré sobre cómo debería realizarse la gestión de paquetes con los gestores NPM, Bower y Gulp.

### NPM - Node Package Manager
Si bien __uno__  de los propósitos de NPM es el desarrollo de aplicaciones Node, no sólo se puede usar para ese fin. Tambien puede utilizarse para gestion de paquetes "Node" (como angular-cli y demas).
En una aplicacion ASP.Net Core se puede utilizar Node para obtener dependencias del lado del cliente, como por ejemplo _Angular_, _JQuery_ o _Bootstrap_; pero tambien se puede obtener los beneficios de herramientas que automatizan el trabajo como _Grunt_ o _Gulp_.
Para tener una estructura mas "ordenada", vamos a utilizar NPM para instalar herramientas del lado del servidor.

Una vez tengamos nuestro Proyecto NetCore se puede agregar un archivo de Configuración NPM al proyecto:

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-02.png)

Cuando el archivo este en el proyecto, se puede ver la configuración basica que contiene:

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-03.png)

Se puede agregar paquetes NPM al proyecto de dos formas:
 
 * Agregando los nombres y versiones de los paquetes manualmente al archivo __package.json_

 * Instalando los paquetes NPM desde consola, pero con la opcion __-S__ (Esta opción instalará el paquete, pero pondrá una linea en el archivo __package.json__ con el nombre y la versión instalada). Por ejemplo:
~~~terminal
npm install gulp -S
~~~

Una vez se tengan identificados los paquetes a instalar, vemos que ocurre la magia:

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-04.png)

Como se puede ver, cada paquete identificado en el archivo __package.json__ ha sido instalado y configurado como dependencia en el proyecto.

___Nota:___ _Como podrán ver, sólo se ha instalado GULP con NPM. Las librerias del lado del cliente van a instalarse con Bower. Esto con el propósito de tener un orden y clasificación referente a lo que se va autilizar en el lado del cliente y lo que se va a utilizar en el lado del servidor._


### Bower

Vimos que instalar paquetes con NPM es muy sencillo. ¿Cierto?.
Ahora vamos a instalar dependencias del lado del cliente como _Bootstrap_, _JQuery_ y demas. Para ello, primero se debe agregar un archivo de configuración Bower en el Proyecto:

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-05.png)

Cuando el archivo este en el proyecto, se puede ver la configuración basica que contiene:

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-06.png)

Al igual que NPM, se puede agregar paquetes desde BOWER al proyecto de dos formas:

 * Agregando los nombres y versiones de los paquetes manualmente al archivo __bower.json__

 * Instalando los paquetes BOWER desde consola, pero con la opcion __--save__ (Esta opción instalará el paquete, pero pondrá una linea en el archivo __bower.json__ con el nombre y la versión instalada). Por ejemplo:
~~~terminal
bower install jquery --save
~~~

Una vez se tengan identificados los paquetes a instalar, vemos que con __Bower__ tambien ocurre la magia:

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-07.png)

Como se puede ver, cada paquete identificado en el archivo __bower.json__ ha sido instalado y configurado como dependencia en el proyecto.

Por defecto, Bower instala todo en la carpeta __wwwroot/lib__, aunque se puede especificar una nueva ruta en el archivo __bower.json__ con:

~~~json
{
    "directory" : "wwwroot/lib"
}
~~~

### Gulp y Bundles

Hasta ahora ya tenemos instalado Gulp (con NPM) y todas las librerias necesarias del lado del cliente (con Bower). 
Normalmente lo que sigue es referenciar las librerias del lado del cliente en el __"_Layout"__ principal del proyecto, pero esto no es aconsejable debido a que no es lo más optimo. Aqui es donde entra GULP para crear hacer el "Bundling" y minimizar las referencias (archivos CSS y JS) al maximo.

 * "Bundling": Es un termino que significa combinar multiples archivos en uno solo. La idea tras de esto es minimizar la cantidad de peticiones al servidor, con el fin de mejorar el rendimiento del primer cargado de la pagina.

 * Minimización: Puede aplicarse a diferentes metodos de optimización de código para reducir el tamaño de los archivos (CSS, JS e incluso imagenes). Una práctica comun de "minimización" es acortar los nombres de las variables a una unica letra.  

Para aplicar "Bundling" y minimización en el proyecto, es necesario el archivo __bundleconfig.json__ que por lo general viene por defecto en los proyectos NetCore de tipo ASP:Net. Un ejemplo de este archivo es:

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-08.png)

Como se puede ver, es un archivo JSON que contiene una configuración básica de cómo realizar el Bundling y la minimización:

~~~json
[
  {
    "outputFileName": "wwwroot/css/integrate.min.css",
    "inputFiles": [
      "wwwroot/css/sb-admin-2.css",
      "wwwroot/lib/metisMenu/dist/metisMenu.css",
      "wwwroot/lib/bootstrap-table/dist/bootstrap-table.css"
    ],
    "minify": {
        "enabled": true
    }
  },
  {
    "outputFileName": "wwwroot/js/integrate.min.js",
    "inputFiles": [
      "wwwroot/js/sb-admin-2.js",
      "wwwroot/lib/bootstrap-table/dist/bootstrap-table.js"
    ],
    "minify": {
      "enabled": true,
      "renameLocals": true
    },
    "sourceMap": false
  }
]
~~~

Se tiene dos bloques: el primero para archivos CSS y el segundo para archivos JS.
Se puede ver que ambos configuran un conjunto de archivos en entrada (_inputFiles_), y el archivo de salida que se generará (_outputFileName_).

Ahora, VisualStudio nos ofrece la opción de crear nuestro archivo GULP a partir del archivo __bundleconfig.json__. Para ello, se hace click derecho sobre el archivo y se selecciona la opción "Bundler & Minifier" y luego "Convert to Gulp":

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-09.png)

Esto no sólo creará el archivo __gulpfile.js__, si no que tambien instalará los paquetes necesarios del lado del servidor (con NPM), y los adicionará automaticamente al archivo __packages.json__.

Para realizar un ensayo de lo que hará nuestro proceso de "bundling" y minificación, hacemos click derecho sobre __gulpfile.js__ y seleccionamos la opción "Explorador del ejecutador de Tareas", para ver el siguiente panel:

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-10.png)

Dentro de esta ventana, en la parte izquierda se puede ver un listado de "Tareas". Cada una de estas tereas realiza un trabajo distinto y se puede ir probando de una en una. Sin embrago la más interesante es la que indica __"min"__: Esta tarea realiza el "bundling" y minimización de todos los archivos especificados en  __bundleconfig.json__ para generar los archivos compresos. Para ejecutar la tarea, se hace click derecho sobre la misma y se selecciona "Ejecutar":

Si todo salió bien, se verifica la ruta donde se supone tienen que estar los archivos de salida:

![_config.yml](/images/2017-08-17-Npm-Bower-y-Gulp-en-ASP-NetCore-11.png)

Lo unico que resta es referencia este archivo simplificado en nuestras vistas.
