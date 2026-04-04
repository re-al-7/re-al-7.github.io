---
title: "Integración Jenkins con MSBuild e IIS"
slug: "integracion-jenkins-iis"
date: 2017-06-20
tags: ["integracion continua"]
---

Estos dias me he planteado realizar la implementacion de CI (Integración Continua) en mis proyectos. Para ello he acudido a Jenkins.

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-01.png)

### Jenkins

[Jenkins](https://jenkins.io/) nos permite, de una manera facil e intuitiva, programar el despliegue de nuestras aplicaciones.
Verán que instalarlo es muy simple. [Aquí](https://jenkins.io/doc/book/getting-started/installing/) las instrucciones para instalarlo en cualquier S.O. En mi caso, voy a usarlo sobre Windows Server.

Por defecto, el puerto por el que escucha Jenkins es el 8080, pero podemos modificarlo para escuchar cualquier otro puerto.
Esta modificación se realiza en el archivo **"jenkins.xml"** que se encuentra en el directorio raiz de la instalación de Jenkins

En mi caso, utilizare el puerto 82. La configuración debera quedar asi:

~~~xml
<arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -jar "%BASE%\jenkins.war" --httpPort=82 --webroot="%BASE%\war"</arguments>
~~~

Para que la configuración tenga efecto tuve que reiniciar el servicio a través del "Administrador de Servicios" de Windows.

Una vez se tenga todo listo, debe procederse a instalar los plugins recomendados por Jenkins, y después debe instalarse los siguientes plugins a través de "Administrar Jenkins"->"Administrar Plugins":

 * Bitbucket Build Status Notifier Plugin
 * Bitbucket Plugin
 * MSBuild Plugin
 * MSTest plugin

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-02.png)

Ahora procedemos a configurar nuestro MSBuild. Esta herramienta nos permitirá compilar proyectos .Net.

### Configurar MSBuild y Git

Para realizar estas configuraciones, primero debemos tener instaladas estas herramientas en el servidor de despliegue (aka. donde está Jenkins):

 * [MsBuild](https://www.microsoft.com/es-ar/download/details.aspx?id=48159)
 * [Git](https://git-scm.com/)

Una vez tengamos instalados estos utilitarios, ingresamos a "Administrar Jenkins"->"Global Tool Configuration" para ajustar el PATH de cada ejecutable:

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-03.png)

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-04.png)

Podemos crear varias instancias de MsBuild, de acuerdo a la version del mismo. En mi caso estoy utilizando la versión 14, correspondiente a VisualStudio 2015.

### Crear Tareas

Una vez tengamos todo configurado, podemos crear nuestra primera tarea, haciendo click en la barra lateral izquierda.

Ponemos un nombre a la tarea (yo prefiero poner el nombre de mi Solución .Net) y elegimos la opcion **"Crear un proyecto estilo libre"** y presionamos el boton **"OK"**:

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-05.png)

Ahora realizamos la configuración del Proyecto. 

En mi caso voy a elegir, como origen del codigo fuente, un repositorio git en BitBucket:

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-06.png)

Al configurar un repositorio en BitBucket, me pedirá que cree mis credenciales de acceso Git.

Tambien puedo configurar (si tengo el plugin de BitBucket), que se ejecute una "Build" cada vez que se haga un PUSH al repositorio.

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-07.png)

Si marcamos esta opcioón, debemos configurar un **WebHook** en el repositorio de BitBucket apuntando hacia nuestro servidor de despliegue:

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-08.png)

La URL del WeebHook en BitBucket debe ser:

~~~
http://USUARIO_JENKINS:PASSSWORD_JENKINS@SERVIDOR_JENKINS:PUERTO_JENKINS/bitbucket-hook/
~~~

Ahora, volviendo a la configuración de la Tarea en Jenkins, pasamos a la parte mas importante:

En el bloque de **Ejecutar** podemos añadir varias acciones:

#### 1. Restaurar paquetes Nuget de la Solucion

Antes de empezar a realizar el despliegue de nuestros proyectos, es necesario restaurar los paquetes Nuget utilizados en la solución.
Si bien existen plugins para Jenkins que nos ayudan con esto, es mejor realizar la tarea a través de un paso de tipo **"Ejecutar un comando de Windows"**

Obviamente, esto implica que tenemos que tener instalado el ejecutable **"Nuget.exe"** en nuestro servidor de despliegue.

~~~
"C:\Program Files\Nuget\"nuget.exe restore ARCHIVO_DE_SOLUCION.sln
~~~

#### 2. Ejecutar el MsBuild de Toda la solución

Para compilar toda la solución, agregamos un nuevo paso y elegimos la opcion **"Build a VisualStudio project or solution using MsBuild"**. Despues, configuramos el paso de la siguiente forma:

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-09.png)

En el campo "MSBuild File" se debe especificar el archivo que representa la solución (.sln).


#### 3. Publicar el Proyecto principal

Para este paso agregamos una opción **"Build a VisualStudio project or solution using MsBuild"** y especificamos el Path al proyecto principal de nuestra solucion:

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-10.png)

Ademas ponemos el siguiente argumento:
~~~
/T:Build;Package /p:Configuration=DEBUG /p:OutputPath="C:\JenkinsBuilds\NOMBRE_PROYECTO_PRINCIPAL" /p:DeployIisAppPath="/Default Web Site/NOMBRE_SITIO" /p:VisualStudioVersion=14.0
~~~


#### 4. Desplegar al Servidor IIS

Por ultimo, agregamos otro paso de tipo **"Ejecutar un comando de Windows"** y especificamos el siguiente comando:

~~~
xcopy "C:\JenkinsBuilds\NOMBRE_SOLUCION\_PublishedWebsites\NOMBRE_PROYECTO_PRINCIPAL" /O /X /E /H /K /Y /d "C:\inetpub\wwwroot\NOMBRE_SITIO_EN_IIS\"
~~~

Esta instrucción copiará los archivos recurrentemente, pero sólo aquellos que han cambiado recientemente


#### 5. Compilar AHORA y ver la tendencia de tiempo de ejecución

Una vez esté todo configurado, podemos ordenar a **Jenkins** realizar la compilación y verificar las estadísticas:

![_config.yml](/images/2017-06-20-Integracion-Jenkins-IIS-11.png)