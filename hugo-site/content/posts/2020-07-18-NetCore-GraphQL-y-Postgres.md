---
title: "Documentar mi Base de Datos con SchemaSpy"
date: 2020-07-18
tags: ["base de datos", "documentacion"]
---


Recientemente conoci la herramienta SchemaSpy para documentar Bases de Datos, generando documentación visual, detallada, pero sobretodo, muy entendible.


![_config.yml](/images/2020-07-18-Documentar-mi_Base-de-Datos-con-SchemaSpy.png)

Esta entrada servirá como introducción para ver lo que podemos hacer con [SchemaSpy](http://schemaspy.org/) y su uso en nuestros proyectos. Lo primero, descargar e instalar SchemaSpy, para ellos seguimos los pasos descritos en su [documentación oficial](https://schemaspy.readthedocs.io/en/latest/installation.html).

[SchemaSpy](http://schemaspy.org/) funciona con la mayoría de los motores de Base de Datos, como ser Oracle, MS-SQL, DB2, PostgreSQL, MySQL, etc; y se conecta a ellos a través de JDBC.

La forma más simple de ejecutar [SchemaSpy](http://schemaspy.org/) es a través de linea de comandos, pasando como parámetros los datos para conexión a la Base de Datos que queremos documentar:

~~~bash
java -jar schemaspy.jar -t mssql05 -dp C:/sqljdbc4-3.0.jar -db NOMBRE_BD -host SERVIDOR -port 1433 -s dbo -u USUARIO_BD -p PASSWORD_BD -o DIRECTORIO_DE_SALIDA
~~~

Pero si queremos tener guardada la configuración utilizada para documentar la Base de Datos de uno de nuestros proyectos, es mejor utilizar un archivo de configuración normalmente denominado "schemaspy.properties". A continuación un detalle de cómo se vé uno:

~~~bash
#
#java -jar .\schemaspy-6.1.0.jar -configFile schemaspy.properties
#

# database type (pgsql, mssql, ora, mysql)
schemaspy.t=pgsql
 
# path to the database JDBC driver (postgresql-42.2.14.jar, mssql-jdbc-8.2.2.jre8.jar, ojdbc6.jar, mysql-connector-java-8.0.21.jar)
schemaspy.dp=./drivers/postgresql-42.2.14.jar

schemaspy.host=127.0.0.1
schemaspy.port=5432
 
# database name
schemaspy.db=db_prueba
 
# database user
schemaspy.u=postgres
schemaspy.p=**********
 
# output folder for the generated resukt
schemaspy.o=C:/mi_documentacion
 
# database schema
schemaspy.s=public
~~~

Para utilizar este archivo como parámetro para SchemaSpy, debemos ejecutar

~~~bash
java -jar schemaspy.jar -configFile schemaspy.properties
~~~

SchemaSpy genera el resultado en archivos HTML, JS, CSS e imágenes (generadas con Graphviz), y todo es guardado en la carpeta de salida "schema.o".

La documentación incluye todas las tablas, vistas, columnas, constraints, procedimientos almacenados, funciones y más detalles de cada tabla. Además se genera una representación visual del diagrama relacionales de las tablas en la Base de Datos:

![_config.yml](/images/2020-07-18-Documentar-mi_Base-de-Datos-con-SchemaSpy-1.png)

Esta herramienta es muy util para documentar nuestra Base de Datos o para verificar la estructura y problemas de Bases de Datos que nos encontramos en nuetsros distintos proyectos.