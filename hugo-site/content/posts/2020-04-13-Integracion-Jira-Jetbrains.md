---
title: "Integrar JIRA con los IDEs de Jetbrains"
date: 2020-04-13
tags: ["desarrollo"]
---

Hoy vamos a ver como se puede configurar la integración de los Issues creados en JIRA para verlos como *Tasks* en los IDEs de Jetbrains (especificamente en Rider).


![_config.yml](/images/2020-04-13-Integracion-Jira-Jetbrains.png)

Primero se debe configurar los *Servidores* desde el IDE. Para ello ingresamos a *Tools | Task & Contexts | Configure Servers...*.


![_config.yml](/images/2020-04-13-Integracion-Jira-Jetbrains-01.png)

Luego se agrega el Servidor de tipo *Jira*:

![_config.yml](/images/2020-04-13-Integracion-Jira-Jetbrains-02.png)

En el cuadro de información general se debe colocar la siguiente información:

 - *Server URL:* Es donde esta alojado nuestro servidor JIRA. Por ejemplo _https://xxxxxxxx.atlassian.net_

 - *Email:* Es la dirección de nuestro correo electrónico de usuario de JIRA

 - *API Token:* Es el token de acceso provisto por Jira. Este Token se puede obtener desde [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens).

 - *Search:* Es el filtro que se utilizará para mostrar los Issues en el pane de Tareas _Tasks_. El filtro recomendado es: project = "NOMBRE_PROYECTO" and assignee = currentUser() and status != Done order by updated

Presionamos el botón *Test* para verificar que nuestra configuración es correcta:

![_config.yml](/images/2020-04-13-Integracion-Jira-Jetbrains-03.png)

Se pueden 

Con ello, nuestra integración de Rider con Jira ya está terminada.

Se pueden ver los _issues_ creados desde el propio IDE, haciendo click en la opción *Tools | Task & Contexts | Open Task...*:


![_config.yml](/images/2020-04-13-Integracion-Jira-Jetbrains-04.png)

Despueés de realizar la respectiva sincronización se nos mostrará el detalle de Issues en JIRA (que no estén en estado FInalizado _DONE_):

![_config.yml](/images/2020-04-13-Integracion-Jira-Jetbrains-05.png)

Al abrir un nuevo _issue_ se nos mostrará la siguiente pantalla:

![_config.yml](/images/2020-04-13-Integracion-Jira-Jetbrains-06.png)

Esta ventana nos permite configurar el issue abierto:

 - *Update issue state* permite modificar el estado del issue. Este cambio se reflejará en JIRA

 - *Create changelist* creacrá un listado de cambios dentro del IDE, que podra ser centralizado via GIT al repositorio central

 - *Shelve current changes* si ésta opción está marcada, se dejará de lado los cambios actuales

 - *Use branch* si se desea crear una rama en el repositorio.

Una vez abierto el issue, podemos trabajar en los cambios, correcciones y mejoras relacionados.

Cuando se desee _cerrar_ la tarea, se elige la opción *Tools | Task & Contexts | Close active Task...*, donde se podrá definir el comportamiento del issue en JIRA:


![_config.yml](/images/2020-04-13-Integracion-Jira-Jetbrains-07.png)

En la ventana se puede elegir el unevo estado del issue, y además asociar un COMMIT.

Con todo ello, podemos gestionar nuestro contenido de JIRAa desde el IDE; una ventaja muy util que nos da Jetbrains.