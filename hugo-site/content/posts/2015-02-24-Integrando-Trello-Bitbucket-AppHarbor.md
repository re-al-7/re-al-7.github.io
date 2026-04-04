---
title: "Integrando Trello + Bitbucket + AppHarbor"
date: 2015-02-24
tags: ["integracion continua"]
---

En éste tutorial se va a mostrar cómo pueden enlazarse tres servicios para lograr una infraestructura que permita automatizar varias tareas en la cadena de desarrollo de software.

Primero necesitamos tener instalado Git, además de tener un repositorio localmente (en nuestro equipo) para alojarlo remotamente en BitBucket. 

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-01.png)

Una vez tengamos el repositorio en BitBucket, nos dirigimos a AppHarbor para crear nuestra aplicacion. 

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-02.jpg)

Una vez se haya creado la aplicación en AppHarbor, se debe elegir la opcion de despliegue desde BitBucket ("Configure BitBucket to deploy to AppHarbor"). 

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-03.jpg)

Cuando presionemos ése enlace, se nos pedir conectar nuestra cuenta de BitBucket, y acto seguido, nos pedirá el repositorio que queremos vincular 

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-04.png)

Si todo va bien, en BitBucket podremos ver que se han añadido permisos a nuestro repositorio: 

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-05.png)

Y tambien se ha añadido automáticamente un Hook 

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-06.png)

Y listo!!! Cada vez que hagamos un PUSH desde nuestro proyecto local, se realizará una compilación en AppHarbor, poniendo a nuestra disponibilidad el proyecto compilado: 

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-07.jpg)

Si el proyecto es una aplicación de escritorio, se creará un enlace de descarga del Ejecutable y todas sus dependencias. En cambio, si se trata de un proyecto Web (o WCF), se creará un directorio donde puede probarse la aplicación: 

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-08.jpg)

Todo éste proceso automatiza el despliegue de aplicaciones, gracias a los WebHooks que se tienen a la mano en BitBucket.

Y es precisamente con éstos WebHooks que se va arealizar la integración con Trello. Podemos crear un Hook de tipo Mail en BitBucket de tal manera que se envie un correo a la dirección de nuestro tablero en Trello. 

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-09.png)

Personalmente configuro todos mis PUSH para que se "inserten" en la lista de "Testing" de mi tablero.

![_config.yml](/images/Integrando-Trello-Bitbucket-AppHarbor-10.png)