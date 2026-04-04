---
title: "Documentar tu proyecto ASP.Net y mostrarlo como un formulario mas"
date: 2017-06-17
tags: ["documentacion"]
---

Ahora mostraré como puedes hacer que tu proyecto WebForms ASP.Net genere cada vez su propia documentación en base a los comentarios del código fuente y además se muestre como un formulario.

### Para empezar
Para ello utilizaremos dos recursos:

 * [VsXMd](https://github.com/lijunle) de [lijunle](https://github.com/lijunle)
 * [CommonMark.Net](https://github.com/Knagis/CommonMark.NET) de [Knagis](https://github.com/Knagis)

Estos dos proyectos los podemos obtener a traves de su respectivo repositorio en GitHub o a traves de Nuget.

Para nuestro propósito, vamos a trabajar con el Codigo Fuente de VsXMd y con el paquete Nuget de CommonMark.Net

### Generando la documentación en XML

Como bein sabemos, VisualStudio tiene la posibilidad de generar la documentación de nuestro proyecto en un archivo XML

La documentación se genera en base a los comentarios que se realicen sobre los métodos, propiedades y atributos de las clases de nuestro proyecto.

Si no conoces cómo generar esta documentación, te sugiero que te pases por [aqui](https://msdn.microsoft.com/es-es/library/x4sa0ak0(v=vs.100).aspx)

### Generando la documentación en Markdown

Lo primero que tenemos que hacer es abrir de manera separada el proyecto **VsXMd**.
Una vez abierto, podremos compilarlo y generar sus respectivos ejecutables, tal como muestra en la figura:

![_config.yml](/images/2017-06-17-Documentar-proyecto-asp-net-01.png)

Necesitamos tener el ejecutable (y todas sus dependencias) por una razon:vamos a utilizar este ejecutable para generar un Script PostCompilacion en nuestro proyecto que queremos documentar

Además, la razon por la que trabajé con el codigo fuente (y no con el paquete Nuget), fue por que con el codigo fuente tengo la libertad de poder elegir lo que quiero que aparezca en mi documentacion. Asi por ejemplo, no me interesa poner en la documentación los nombres y las descripciones de todos los controles.

Para hacer este cambio, modifiqué el metodo: **private static IEnumerable<IUnit> ToUnits(XElement docElement)** en la clase **Converter.cs** del proyecto **Vsxmd** para que quede asi:

~~~csharp
private static IEnumerable<IUnit> ToUnits(XElement docElement)
{
    // assembly unit
    var assemblyUnit = new AssemblyUnit(docElement.Element("assembly"));

    // member units
    var memberUnits = docElement
        .Element("members")
        .Elements("member")
        .Select(element => new MemberUnit(element))
        .Where(member => member.Kind != MemberKind.NotSupported && member.Kind != MemberKind.Constants)
        .GroupBy(unit => unit.TypeName)
        .Select(MemberUnit.ComplementType)
        .SelectMany(group => group)
        .OrderBy(member => member, MemberUnit.Comparer);

    // table of contents
    var tableOfContents = new TableOfContents(memberUnits);

    return new IUnit[] { tableOfContents }
        .Concat(new[] { assemblyUnit })
        .Concat(memberUnits);
}
~~~

Si se fijan en la clausula **Where** (y la comparan con la que originalmente esta en el proyecto de lijunle), podrán ver que estoy excluyendo los elementos de tipo **Constants**.

Antes:
~~~csharp
.Where(member => member.Kind != MemberKind.NotSupported)
~~~
Despues:
~~~csharp
.Where(member => member.Kind != MemberKind.NotSupported && member.Kind != MemberKind.Constants)
~~~

Una vez realizado este cambio, compilo nuevamente el proyecto para obtener el nuevo archivo ejecutable

### Creando el Script PostCompilación en nuestro proyecto

Ahora nos vamos a **nuestro** proyecto para añadir los scripts post compilación.
Primero debemos copiar el ejecutable y las DLLs del proyecto **VsXMd** a la carpeta de nuestra Solucion para que quede mas o menos asi:

![_config.yml](/images/2017-06-17-Documentar-proyecto-asp-net-02.png)

Una vez tengamos copiados estos archivos, debemos ir a las propiedades de nuestro **Proyecto**, y nos vamos a la ficha de **Eventos de Compilación**:

![_config.yml](/images/2017-06-17-Documentar-proyecto-asp-net-03.png)

En la **"Linea de comandos del evento posterior a la compilación"** colocamos:

~~~
"$(SolutionDir)VsXMd"\Vsxmd.exe "$(ProjectDir)App_Data\XmlDocument.xml" "$(ProjectDir)App_Data\XmlDocument.md"
~~~

En mi caso, cambie la ruta (y el nombre) del archivo XML generado por el VisualStudio, para colocarlo en la carpeta **App_Data**

Ahora compilamos nuestro Proyecto y verificamos que el archivo Markdown existe en la ruta especificada. Cada vez que compilemos nuestro proyecto, se generará/actualizará la documentación XML y su respectivo archivo MD


### Creando el formulario para mostrar la documentación en HTML a partir del Markdown

Finalmente, podemos hacer que ese archivo Markdown sirva como fuente para uno de nuestros formularios. Aqui es donde utilizaremos el paquete **CommonMark** para interpretar texto en *Markdown* y representarlo en *HTML*

En éste proyecto de ejemplo, trabajé con WebForms y Bootstrap. Dentro de un nuevo WebForm (archivo aspx) creo el siguiente bloque de código:

~~~html
<div class="row">
    <div class="col-lg-12 col-md-12">
        <div runat="server" id="divDocs"></div>
    </div>
</div>
~~~

En el CodeBehind, en el evento **Page_Load** tenemos

~~~csharp
protected void Page_Load(object sender, EventArgs e)
{
    if (!Page.IsPostBack)
    {
        //El Markdown
        var strMarkDown = Server.MapPath("~/App_Data/XmlDocument.md");
        string text = File.ReadAllText(strMarkDown);
        divDocs.InnerHtml = CommonMarkConverter.Convert(text);
    }
}
~~~

Y listo!!! Este seria el resultado:

![_config.yml](/images/2017-06-17-Documentar-proyecto-asp-net-04.png)

![_config.yml](/images/2017-06-17-Documentar-proyecto-asp-net-05.png)

Si queremos hilar mas fino, podemos modificar el proyecto **VsXMd** para escribir los textos en español, o para modificar lo que queremos que se muestre.

Espero les sirva.