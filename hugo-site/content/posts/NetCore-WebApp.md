---
title: "Guía rapida para scaffolding de aplicaciones web con NetCore y PostgreSql"
date: 2021-06-22
tags: ["desarrollo"]
---


Este post es un poco diferente a los que habitualmente se tienen aqui. Ahora vamos a realizar una guia rápida para preparar el desarrollo de una aplicación web **MVC** en NetCore con Conexion a PostgresSql via EntityFramework.


![_config.yml](/images/2021-06-22-NetCore.png)


## Requisitos

Instalar o actualizar la herramientas de Entity Framework

~~~terminal
dotnet tool install --global dotnet-ef

dotnet tool update --global dotnet-ef
~~~


Instalar o actualizar la herramienta de scaffolding:

~~~terminal
dotnet tool install --global dotnet-aspnet-codegenerator

dotnet tool update --global dotnet-aspnet-codegenerator
~~~

Añadimos el certificado SSL en el PATH del proyecto

~~~terminal
dotnet dev-certs https --trust
~~~


## Inicio del proyecto

Añadir los paquetes Nuget:

 - Npgsql
 
 - Microsoft.EntityFrameworkCore

 - Microsoft.EntityFrameworkCore.SqlServer.
 
 - Npgsql.EntityFrameworkCore.PostgreSQL
 
 - Microsoft.VisualStudio.Web.CodeGeneration.Design
 
 - Microsoft.EntityFrameworkCore.Tools
 
 - Microsoft.AspNetCore.Identity.UI
 

Después de añadir las referencias del proyecto, se debe ejecutar un BUILD.

Ahora se puede crear los modelos con la herramienta [dotnet ef](https://docs.microsoft.com/en-us/ef/core/miscellaneous/cli/dotnet), desde linea de comandos, desde el PATH de la solución:

~~~powershell
 dotnet ef dbcontext scaffold "Host=localhost;Database=*NOMBRE_BD*;Username=*USUARIO_BD*;Password=*PASSWORD_BD*" Npgsql.EntityFrameworkCore.PostgreSQL -o Models -c ReAlDbContext --context-dir . --no-build --force --startup-project *NOMBRE_PROYECTO*
~~~

En el archivo *appsettings.json*, se debe añadir el segmento "ConnectionStrings":

~~~json
{
  "ConnectionStrings": {
    "DataAccessPostgreSqlProvider": "User ID=*USUARIO_BD*;Password=*PASSWORD_BD*;Host=localhost;Port=5432;Database=*NOMBRE_BD*;Pooling=true;"
  }, 
  [.....]
}

~~~

En el archivo *Startup.cs*, modificar el método *ConfigureServices*:

~~~csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
	// [ReAl] Usar una base de datos PostgreSQL
	var sqlConnectionString = Configuration.GetConnectionString("DataAccessPostgreSqlProvider");
	services.AddDbContext<ReAlDbContext>(options =>
		options.UseNpgsql(
			sqlConnectionString,
			b => b.MigrationsAssembly(typeof(ReAlDbContext).Assembly.FullName)
		)
	);
	
	// [ReAl] Para acceder al archivo de configuracion
	services.AddSingleton<IConfiguration>(Configuration);

	services.AddControllersWithViews();	
}
~~~


Ahora ya se puede crear las paginas para los ABMs (CRUD) con la creación automática (scaffolding) de los CONTROLLERS, realizados a partir de los MODELOS:

~~~powershell
dotnet aspnet-codegenerator controller --project . --force --controllerName *NOMBRE_TABLA_Controller* --model *NAMESPACE_MODELO*.*NOMBRE_MODELO* --dataContext ReAlDbContext --relativeFolderPath .\Controllers\ --useDefaultLayout --referenceScriptLibraries 
~~~

Tambien se puede crear solo vistas (Empty|Create|Edit|Delete|Details|List):

~~~terminal
dotnet aspnet-codegenerator view myDetails Details -outDir Views/*NOMBRE_TABLA* --project . --model *NAMESPACE_MODELO*.*NOMBRE_MODELO* --dataContext ReAlDbContext --useDefaultLayout --referenceScriptLibraries --force
~~~

Y por ultimo, si se ve necesario reconstruir un modelo, se puede usar:

~~~terminal
dotnet ef dbcontext scaffold "Host=localhost;Database=*NOMBRE_BD*;Username=*USUARIO_BD*;Password=*PASSWORD_BD*" Npgsql.EntityFrameworkCore.PostgreSQL -o Models -c ReAlDbContext_new --context-dir . --no-build --force -t enc_informantes
~~~

## Personalizando el Scaffolding

Se puede crear los propios templates creando la carpeta *Templates* y añadiendo el siguiente fragmento al archivo csproj

~~~xml
<ItemGroup>
    <Content Update="appsettings.json">
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
    <Content Remove="Templates\**">
    </Content>
</ItemGroup>
~~~  

Se puede copiar los templates originales desde:

~~~terminal
C:\Users\{USUARIO}\.nuget\packages\microsoft.visualstudio.web.codegenerators.mvc\3.1.1\Templates
~~~

Y a partir de alli, modificarlos según necesidad