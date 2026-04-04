---
title: "Tests con JMeter Recording Controller"
slug: "JMeter-Recording-Controller"
date: 2021-06-29
tags: ["test"]
---

En el anterior post usabamos **Selenium** para grabar la funcionalidad de una pag;ína web y luego automatizar las pruebas. En esta ocasión vamos a usar **JMeter Recording Controller** para grabar las peticiones HTTP que se realizan a un sitio y con ello simular la navegación de un usuario.


![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-01.png)

Este tipo de escenarios nos puede servir para realizar pruebas de carga o estrés a nuestro servidor de aplicaciones. Al ejecutar peticiones HTTP, sin necesidad de una UI (como chromiun o gecko), podemos enviar varios ciclos de peticiones al mismo tiempo, sin necesidad de ver afectada la memoria de nuestro cliente.

Primero necesitamos instalar [JMeter](https://ci-builds.apache.org/job/JMeter/job/JMeter-trunk/lastSuccessfulBuild/artifact/src/dist/build/distributions/) y crear un nuevo proyecto a partir de un _Template_:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-02.png)

En la ventana emergente, elegimos la opción `Recording with Think Time` y presionamos el botón `Create`:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-03.png)

Este template nos creará la siguiente estructura de proyecto

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-04.png)

Ahora añadimos los siguientes elementos al proyecto, dentro del elemento `Thread Group` añadir:

~~~terminal
Add &rarr; Listener &rarr; Summary Report
Add &rarr; Listener &rarr; View Results Tree
Add &rarr; Listener &rarr; Simple Data Writer
~~~

Tambien, dentro de `TestPlan` añadimos:

~~~terminal
Add &rarr; Config Element &rarr; CSV DataSet Config
~~~

Con todo ello, la estructura de nuestro proyecto debe verse asi:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-05.png)


Antes de comenzar con la grabación, en el elemento `HTTP(S) Test Script Recorder` debemos definir algunos parametros, como ser: `Port` y `HTTPS domains`, además de un identificador `Transaction Name` que nos sservirá para diferenciar por grupos las peticiones a realizar (En nuestro ejemplo, vamos a usar el texto LOGIN). Debe quedar asi:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-06.png)

Ahora presionamos el botón `Start`. JMeter nos mostrará un mensaje sobre la creación de un Certificado que debe instalarse en nuestro navegador web:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-07.png)

Este certificado se encuentra en el directorio `bin` dentro de la carpeta donde se ejecutó JMeter:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-08.png)

En nuestra prueba, vamos a instalarlo en Mozilla Firefox, para ello vamos al menpú `Ajustes` para importar el nuevo certificado:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-09.png)

Una vez importado el certificado, debemos cambiar la configuración del PROXY de Mozilla Firefox para que todas las solicitudes HTTP sean enviadas a JMeter y puedan ser grabadas:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-10.png)

Con todo esto realizado, podemos comenzar la grabación de nuestro caso de prueba, a través del panel `Recorder: Transactions Control`:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-11.png)

Este panel nos permitirá añadir etiquetas (1), e incluso reiniciar los contadores de las etiquetas (2) para que podamos separar la ejecución de nuestra prueba en etapas.

Al terminar de grabar el caso de prueba, presionamos el boton STOP del panel `Recorder: Transactions Control`. Nos fijamos en el elemento `Recording Controller` de nuestro proyecto en JMeter, y deberíamos tener algo asi:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-12.png)

Al revisar cada petición HTTP realizada en nuestra grabación nos encontraremos con este tipo de detalles sobre datos de formulario:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-13.png)

Estos valores podemos parametrizarlos a través del elemento `CSV DataSet Config` que se encuentra en nuestro proyecto.

Para ello, creamos un archivo CSV (sin cabeceras y separado por comas), que tenga, por ejemplo, los siguientes valores:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-14.png)

- La columna 1 se refiere al nombre de usuario

- La columna 2 se refiere a la contraseña de inicio de sesión del usuario

- La columna 3 se refiere al productId que deseamos añadir a nuestro carrito de compra

- La columna 4 se refiere al SKU del producto seleccionado; esto nos servirá para simular tráfico a través de la URL

En JMeter, la configuración del elemento `CSV DataSet Config` debe quedar mas o menos asi:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-15.png)

Es importante definir los nombres de las columnas en MAYUSCULAS, debido a que eso definirá los nombres de nuestras variables.

Para usar una variable en el script que se ha creado, se usa la siguiente sintaxis:

~~~terminal
${NOMBRE_VARIABLE} 
~~~

Por lo que, nuestra petición de LOGIN, se verá asi:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-16.png)

También se puede usar la nomenclatura de nombre de variables en las URLs:

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-17.png)

Finalmente, para usar todas las variables de nuestro archivo CSV, y que JMeter vaya iterando cada registro, se debe realizar la siguiente configuración en el elemento `Thread group`

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-18.png)

Podemos los resultados en el elemento Summary Report de nuestro proyecto

![_config.yml](/images/2021-06-29-JMeter-Recording-Controller-Phase-19.png)