---
title: "SonarQube para metricas del Codigo en VisualStudio"
date: 2017-06-22
tags: ["integracion continua"]
---

Ultimamente he estado viendo varias herramientas para ver las métricas y estadísticas del codigo fuente de algun proyecto; pero de entre todas estas herramientas destaca SonarQube, por una sencilla razon: es *open source*.

![_config.yml](/images/2017-06-22-SonarQube-para-estadisticas-en-VisualStudio-01.png)

**SonarQube** es una plataforma de código abierto para el análisis de la calidad de nuestro código y para obtener métricas que pueden ayudar a mejorar la calidad del código de cualquiera de nuestros programas. 

Es una herramienta que nos sirve para *auditar* el código dentro de nuestro ciclo de desarrollo de nuestra aplicación. Existen distintos tipos de problemas que se pueden encontrar en un código, éstos pueden clasificarse en 5 grupos según su severidad: Bloqueador, crítico, grave, menor e informativo. 

### Instalando y configurando SonarQube

Primero que necesitamos descargar algunas cosas desde SonarQube:

 * El servidor desde su [respectiva página](https://www.sonarqube.org/downloads/).

 * El Scanner para el MsBuild desde [aqui](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+MSBuild).

 * El SonarQubeScanner desde [aqui](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner).

Una vez se descargue, descomprimimos todo. Yo tengo esta estructura:

![_config.yml](/images/2017-06-22-SonarQube-para-estadisticas-en-VisualStudio-02.png)

Despues agregamos dos variables de entorno a nuestro PATH:

 * La ruta al directorio **sonar-scanner-msbuild**

 * La ruta al directorio **bin** de **sonar-scanner**

En mi caso, sería:

![_config.yml](/images/2017-06-22-SonarQube-para-estadisticas-en-VisualStudio-03.png)

Ahora podemos configurar la conexión a la BD. Yo voy a utilizar un servidor **PostgreSql** (SonarQube también soporta otros tipos de Bases de Datos), por lo que tengo que encontrar el siguiente archivo:

~~~
/PATH_SONARQUBE_SERVER/conf/sonar.properties
~~~

Editando ese archivo, colocamos el nombre de usuario y la contraseña de conexión al servidor:

~~~
# User credentials.
# Permissions to create tables, indices and triggers must be granted to JDBC user.
# The schema must be created first.
sonar.jdbc.username=USUARIO_BD
sonar.jdbc.password=PASSWORD_BD
~~~

Nuevamente, como yo voy a utilizar PostgreSql, descomento la sección correspondiente e introduzco la URL JDBC de conexion:

~~~
#----- PostgreSQL 8.x/9.x
# If you don't use the schema named "public", please refer to http://jira.sonarsource.com/browse/SONAR-5000
sonar.jdbc.url=jdbc:postgresql://IP_SERVIDOR/sonarqube
~~~

Esto quiere decir que debo crear una nueva Base de Datos en mi servidor con el nombre **sonarqube**

Por ultimo, se debe configurar los Scanners descargados. Para ello se busca el archivo **sonar-scanner.properties** en cada uno de los *scaners* (normalmente se encuentran en la carpeta **conf**) y se especifica el servidor SonarQube donde *enviarán* todas las estadísticas. Por defecto, SonarQube *escucha* en el puerto 9000 (aunque esto se puede cambiar):

~~~
#----- Default SonarQube server
sonar.host.url=http://localhost:9000
~~~

Este cambio debe realizarse en todos los Scanners que tengamos descargados

Ahora si, podemos iniciar el Servidor SonarQube. Para ello nos vamos a la carpeta **bin** del directorio donde está el SONARQUBE. Veremos varias carpetas:

![_config.yml](/images/2017-06-22-SonarQube-para-estadisticas-en-VisualStudio-04.png)

Dependiendo del Sistema OPerativo que tengamos, se accede a su respectiva carpeta y se inicia el servicio con el archivo **StartSonar.bat**.

Si quisiéramos instalar SonarQube como servicio, podemos ejecutar **InstallNTService.bat**, y para desinstalarlo: **UninstallNTService.bat**

Accedemos a la direccion **http://localhost:9000/** y podremos ver nuestro servidor SonarQube ejecutándose:

![_config.yml](/images/2017-06-22-SonarQube-para-estadisticas-en-VisualStudio-05.png)

El usuario y contraseña por defecto son **admin/admin**


### Creando el Proyecto en SonarQube y ejecutando el análisis con MsBuild

Antes de ejecutar el scanner sobre nuestro proyecto, es necesario crearlo. Para ello ingresamos a nuestro servidor de SonarQube (http://localhost:9000/), y nos vamos a "Management Projects":

![_config.yml](/images/2017-06-22-SonarQube-para-estadisticas-en-VisualStudio-06.png)

Ahi podemos crear un proyecto especificando el nombre y una palabra clave para identificarlo:

![_config.yml](/images/2017-06-22-SonarQube-para-estadisticas-en-VisualStudio-07.png)

Este nombre y palabra clave será utilizados al momento de realizar el escaneo con **sonar-scanner-msbuild** o con cualquier otro escanner.

Ahora nos dirigimos al directorio raiz de la solución de VisualStudio que deseamos analizar y ejecutamos las siguientes instrucciones:

~~~
MSBuild.SonarQube.Runner begin /key:"SANEAMIENTO" /name:"Segip.Licencias.Saneamiento" /version:1
MsBuild
MSBuild.SonarQube.Runner end
~~~

Para que estos comandos funcionen, la carpeta de **sonar-scanner-msbuild** debe estar en la variable de entorno PATH.

A mi me pasó que ejecutando el comando MsBuild, se ejecutó la versión del .NetFramework 4, y no la del VisualStudio 14.0
Esto hacia que la instruccion me diera errores, por lo que para ejecutar el MSBuild le di todo el PATH:

~~~
C:\Program Files (x86)\MSBuild\14.0\Bin\MSBuild.exe
~~~

Una vez termine el analisis (el tiempo variará dependiendo del tamño del proyecto), podemos acceder al servidor SonarQube y verificar el procesamiento del escaneo:

![_config.yml](/images/2017-06-22-SonarQube-para-estadisticas-en-VisualStudio-08.png)

Cuando la tarea en Background termine, podremos ver las estadisticas del proyecto

![_config.yml](/images/2017-06-22-SonarQube-para-estadisticas-en-VisualStudio-09.png)

