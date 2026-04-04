---
title: "Mostrar archivo PDF alojado en un servidor FTP"
date: 2016-08-06
tags: ["csharp"]
---

Tenemos un repositorio FTP donde varias oficinas en distintos lugares van alojando archivos PDF que generan con información de su respectivo trabajo. Basicamente todos pueden acceder al FTP y consultar los documentos subidos.

Sin embargo, un nuevo requerimiento necesitaba revisar y calificar el documento. Para ello, se necesitaba que el sistema web que se maneja, muestre los archivos PDF desde el FTP (sin necesidad de descargarlos todos al servidor web).

Buscando, pude encontrar soluciones parciales, pero al final llegué a este bloque de código que me permitía visualizar el archivo requerido:

~~~csharp
FileInfo objFile = new FileInfo(filename);
FtpWebRequest request = (FtpWebRequest)WebRequest.Create(new Uri("ftp://" + ftpServerIP + "/" + filename));
request.Credentials = new NetworkCredential(Ftp_Login_Name, Ftp_Login_Password);

FtpWebResponse response = (FtpWebResponse)request.GetResponse();

Stream responseStream = response.GetResponseStream();
StreamReader reader = new StreamReader(responseStream);

byte[] bytes = null;
using (var memstream = new MemoryStream())
{
    reader.BaseStream.CopyTo(memstream);
    bytes = memstream.ToArray();
}

Response.Clear();
Response.ClearHeaders();
Response.ClearContent();
Response.Cache.SetCacheability(HttpCacheability.NoCache);
Response.ContentType = "application/pdf";
Response.AddHeader("Content-Disposition", "inline; filename=" + objFile.Name);
Response.BinaryWrite(bytes);
Response.End();
~~~
