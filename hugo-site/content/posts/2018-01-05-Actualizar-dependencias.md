---
title: "Actualizar dependencias NPM y BOWER"
date: 2018-01-05
tags: ["desarrollo"]
---

Hoy vamos a ver como actualizar las dependencias de un proyecto 

![_config.yml](/images/2018-01-05-Actualizar-dependencias.png)

Esto quiere decir que los archivos "package.json" y "bower.json" van a cambiar, por lo que si deseas, puedes sacar un backup de los mismos.

En el caso de "package.json", es necesario tener el paquete [NPM Check Updates](https://www.npmjs.org/package/npm-check-updates) instalado:

~~~terminal
npm i -g npm-check-updates
~~~ 

Luego se ejecuta las siguientes instrucciones:

~~~terminal
npm-check-updates -u
npm install
~~~ 

Para el caso de "bower.json" necesitamos tener instalado el paquete [Bower Check Updates](https://github.com/se-panfilov/bower-check-updates).

~~~terminal
npm install -g bower-check-updates
~~~ 

Luego ejecutamos las instrucciones para actualizar:

~~~terminal
bower-check-updates -u
bower install 
~~~

Con ello, nuestras dependencias estaran actualizadas a su ultima version.

Espero les sirva