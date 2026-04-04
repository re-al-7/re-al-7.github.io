---
title: "ClosedXML y el uso de plantillas"
date: 2016-04-26
tags: ["csharp"]
---

Para facilitar la tarea de crear reportes o documentos en formato XLS o XLSX, se tiene el paquete ClosedXML. El mismo tiene soporte para el uso de templates.

![_config.yml](/images/2016-04-26-ClosedXML-y-plantillas.jpg)

Lo primero que tenemos que hacer es instalar el paquete Nuget en nuestro proyecto:

~~~
PM> Install-Package ClosedXML
~~~

Después, el bloque de codigo necesario para mostrar un excel a partir de una plantilla es:

~~~csharp
var strTitulo = "Titulo del reporte";
var dtReporte = ObtenerDatosReporte(); //Funcion que devolverá un DataTable

//Definimos la plantilla y la utilizamos con la libreria ClosedXML
var template = Server.MapPath("~/doc_templates/Reporte.xlsx");
using (var wb = new XLWorkbook(template))
{
    //Ponemos algunos valores en el documento
    wb.Worksheets.Worksheet(1).Cell(5, 1).Value = strTitulo;

    //Podemos insertar un DataTable
    wb.Worksheets.Worksheet(1).Cell(9, 1).InsertTable(dtReporte);
    //Aplicamos los filtros y formatos a la tabla 
    wb.Worksheets.Worksheet(1).Table("Table1").ShowAutoFilter = true;
    wb.Worksheets.Worksheet(1).Table("Table1").Style.Alignment.Vertical =
        XLAlignmentVerticalValues.Center;
    wb.Worksheets.Worksheet(1).Columns(2, 2 + dtReporte.Columns.Count).AdjustToContents();

    //Limitamos el ancho de las columnas a 60
    foreach (var column in wb.Worksheets.Worksheet(1).Columns())
        if (column.Width > 60)
        {
            column.Width = 60;
            column.Style.Alignment.WrapText = true;
        }

    wb.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Center;
    wb.Style.Font.Bold = true;

    //Enviamos el archivo al cliente
    Response.Clear();
    Response.Buffer = true;
    Response.Charset = "";
    Response.ContentType = "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
    Response.AddHeader("content-disposition", "attachment;filename=\"" + strTitulo + ".xlsx\"");

    using (var myMemoryStream = new MemoryStream())
    {
        wb.SaveAs(myMemoryStream);
        myMemoryStream.WriteTo(Response.OutputStream);
        Response.Flush();
        Response.End();
    }
}
~~~ 

Para acceder a la documentación del proyecto, pueden ingresar a su [repositorio](https://github.com/closedxml/closedxml)