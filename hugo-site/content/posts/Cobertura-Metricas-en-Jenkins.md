---
title: "Cobertura y Metricas en Jenkins para proyectos .Net"
date: 2017-06-23
tags: ["integracion continua"]
---

Si ya tenemos nuestra solucion de VisualStudio en Jenkins, el siguiente paso es configurar los reportes de CObertura y Metricas

Para ello necesitamos las siguientes herramientas:
 
 * [OpenCover](https://github.com/opencover/opencover/releases)

 * [ReportGenerator](https://github.com/danielpalme/ReportGenerator/releases)

 * [OpenToCoberturaConverter](https://www.nuget.org/packages/OpenCoverToCoberturaConverter). Una vez descargado el paquete Nuget, procedemos a cambiar la extension del archivo de **nupkg** as **zip**. Al descomprimir el archivo zip, podremos encontrar un ejecutable en la carpeta **Tools**. Ese es el ejecutable que necesitamos.

 * [Metics Power Tools](https://www.microsoft.com/en-us/download/details.aspx?id=48213). EN mi caso, la version para VisualStudio 2015.

 *Nota: Todas las herramientas deberas ser descompresas en un directorio (digamos "C:\JenkinsTools\"), cada una con su respectiva carpeta*

![_config.yml](/images/2017-06-23-Cobertura-Metricas-en-Jenkins-02.png)

 Tambien vamos a necesitar algunos plugins de Jenkins:

  * [HTML Publisher](https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin)

  * [VisualStudio Code Metrics](https://wiki.jenkins-ci.org/display/JENKINS/Visual+Studio+Code+Metrics+Plugin)

  * [Cobertura](https://wiki.jenkins-ci.org/display/JENKINS/Cobertura+Plugin)

Ahora procedemos a configurar el Plugin de **CisuaStudio Code Metrics** a través del menú:
*Administrar Jenkins* -> *Global Tool Configuration*
Buscamos el bloque perteneciente al plugin y lo configuramos tal cual se muestra en la captura:

![_config.yml](/images/2017-06-23-Cobertura-Metricas-en-Jenkins-01.png)

Ya tenemos listo los plugins y las herramientas necesarias. 

***NOTA: En mi caso, como no tengo instalado VisualStudio en la maquina de Deployment, instale _VisualStudio Code Metrics_ pero tambien tube que COPIAR todo el contenido de la carpeta de destino "C:\Program Files (x86)\Microsoft Visual Studio 14.0\Team Tools\Static Analysis Tools\FxCop" de mi maquina de desarrollo al servidor de Deployment ***


### Configurando el Proyecto y los reportes

Ahora nos dirigimos a nuestro proyecto de Jenkins y accedemos al panel de Configuración, para añadir los siguientes pasos:

Agregar un paso **Ejecutar un comando de Windows** con la siguiente instrucción:

~~~
"C:\JenkinsTools\opencover\OpenCover.Console.exe" -target:"C:\JenkinsTools\nunit\nunit3-console.exe" -targetargs:"%JOB_NAME%.Tests\bin\Debug\%JOB_NAME%.Tests.dll /framework:net-4.5 /xml:%JOB_NAME%NunitTestResults.xml /nologo /noshadow" -filter:"+[*]* -[%JOB_NAME%.Tests]*" -register:Path64 -hideskipped:Filter -output:%JOB_NAME%Coverage.xml
~~~

![_config.yml](/images/2017-06-23-Cobertura-Metricas-en-Jenkins-03.png)


Agregar otro paso **Ejecutar un comando de Windows** con la siguiente instrucción:

~~~
"C:\JenkinsTools\ReportGenerator\ReportGenerator.exe" -reports:%JOB_NAME%Coverage.xml -targetDir:CodeCoverageHTML
~~~

![_config.yml](/images/2017-06-23-Cobertura-Metricas-en-Jenkins-04.png)


Agregar un paso mas de tipo **Ejecutar un comando de Windows** con la siguiente instrucción:

~~~
"C:\JenkinsTools\opencover_to_cobertura_converter\OpenCoverToCoberturaConverter.exe" -input:%JOB_NAME%Coverage.xml -output:%JOB_NAME%Cobertura.xml -sources:%WORKSPACE%
~~~

![_config.yml](/images/2017-06-23-Cobertura-Metricas-en-Jenkins-05.png)


Finalmente agregamos un paso de Tipo **VS Code Metrics PowerTool exec**. Alli elegimos la herramienta previamente configurada, especificamos la(s) DLL(s) que deseamos se realice el analisis y un nombre para el archivo XML de salida:

![_config.yml](/images/2017-06-23-Cobertura-Metricas-en-Jenkins-06.png)


### Publicación de los reportes

Ahora añadimos pasos al bloque que se ejecuta DESPUES del bloque *Build*:

Primero añadimos un paso de Tipo **Publicar Informes de "Cobertura"** y especificamos el nombre del archivo XML de origen. Aqui es muy importante respectar las Mayusculas y minusculas, porque si no, Jenkins no encuentra el archivo.

~~~
{NOMBRE_DEL_PROYECTO_JENKINS}Cobertura.xml
~~~

![_config.yml](/images/2017-06-23-Cobertura-Metricas-en-Jenkins-07.png)


Ahora añadimos la generación de un reporte HTML basado en el resultado de OpenCover, con los siguientes parametros:

~~~
CodeCoverageHTML   # HTML directorio
index.htm          # Pagina de inicio del Reporte
Code Coverage      # Titulo del Reporte
~~~

![_config.yml](/images/2017-06-23-Cobertura-Metricas-en-Jenkins-08.png)



Y finalmente agregamos el reporte de Metricas de Visual Studio:

![_config.yml](/images/2017-06-23-Cobertura-Metricas-en-Jenkins-09.png)

El nombre **metrics.xml** debe coincidir con el nombre utilizado como salida del paso **VS Code Metrics PowerTool exec** previamente configurado