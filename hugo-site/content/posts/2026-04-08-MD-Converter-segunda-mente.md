---
title: "MD Converter: Tu bandeja de entrada y tus documentos como segunda mente"
slug: "MD-Converter-segunda-mente"
date: 2026-04-08
tags: ["obsidian", "python", "markdown"]
---

  Hay una brecha entre donde vive tu información y donde quisieras pensar con ella. Los correos de Outlook quedan
  atrapados en el cliente. Los PDFs son archivos planos que no se pueden enlazar. Las hojas Excel no conversan con tus
  notas. MD Converter cierra esa brecha: convierte cualquier documento —incluyendo hilos de correo completos— a Markdown
   limpio, estructurado y listo para herramientas como Obsidian.

## El problema que resuelve

  Si usas una herramienta de gestión de conocimiento personal (PKM) basada en Markdown, sabes el dolor: tus correos más
  importantes viven en Outlook, tus reportes en Word, tus datos en Excel. Copiar manualmente ese contenido es tedioso y
  pierde estructura. Lo que necesitas es un conversor que entienda el contexto de cada formato, no solo extraiga texto
  crudo.

  MD Converter es exactamente eso. No es un extractor genérico —es un sistema con reglas específicas por formato que
  produce Markdown listo para indexar, buscar y enlazar.


## Dos formas de usarlo

  Interfaz web local — Un servidor Flask corre en localhost:5000. Arrastrás archivos, los soltás, y en segundos aparecen
   los .md descargables. El watcher de carpeta va un paso más allá: apuntás a una carpeta (por ejemplo, donde Outlook
  guarda archivos .msg) y cualquier archivo nuevo se convierte automáticamente, sin intervención.

##  CLI — Para uso programático o por lotes:

  python convert_to_md.py reporte.pdf
  python convert_to_md.py correo.msg -o md_output/
  python convert_to_md.py carpeta_de_correos/   # convierte todo
  python convert_to_md.py https://ejemplo.com


## Formatos soportados

| Formato | Librería | Qué produce |
|---|---|---|
| .docx | mammoth | Markdown con encabezados y listas preservados |
| .pdf | pdfplumber | Texto extraído por página |
| .pptx | python-pptx | Diapositivas como secciones Markdown |
| .xlsx / .csv | pandas + tabulate | Tablas Markdown |
| .html / URL | html2text + BeautifulSoup | Markdown con tablas |
| .eml | Biblioteca estándar | Correo con frontmatter YAML |
| .msg | extract-msg | Correo Outlook con soporte de hilos |


  ## El módulo de correos: donde está la mayor inteligencia

  El caso más complejo —y más valioso— es el de los correos de Outlook. Un .msg típico no es un mensaje: es un hilo
  entero de respuestas encadenadas. MD Converter lo descompone en mensajes individuales, cada uno con su propio archivo
  .md y metadatos precisos.

  ## Separación de hilos (converters/email/thread.py)

  El módulo _split_thread reconoce cinco patrones de separador distintos, cubriendo los formatos más comunes que generan
   Outlook Desktop, Outlook Web y Gmail:

  1. ________________________   (línea de subrayados — Outlook Desktop)
  2. On [fecha] [persona] wrote:   (Gmail / Outlook Web)
  3. --- Original Message ---   (clientes alternativos)
  4. Bloque De: + Enviado: standalone   (texto plano)
  5. **From:** + **Sent:**   (cuerpo HTML convertido por html2text)

  Si ningún patrón coincide, el módulo cae en un modo de fallback que detecta bloques citados con > al estilo de
  clientes Unix. El resultado es siempre una lista de segmentos con cuerpo limpio y metadatos propios.

  Cada segmento recibe su fecha real —no la del correo más reciente del hilo, sino la del mensaje original. Esto es
  fundamental para que los archivos queden ordenados correctamente en el sistema de notas.

  ## Limpieza de ruido (_clean_msg_segment)

  Los correos corporativos vienen cargados de disclaimers, avisos automáticos de Microsoft, firmas redundantes y URLs de
   imágenes embebidas que no tienen sentido fuera del cliente. _clean_msg_segment aplica una lista de patrones de ruido
  configurables y los elimina antes de escribir el archivo:

~~~
  NOISE = [
      re.compile(r'No suele recibir correo electrónico de', re.IGNORECASE),
      re.compile(r'This e-?mail and any attachments? are confidential', ...),
      re.compile(r'Imprime sólo si es necesario', ...),
      # ... más patrones
  ]
~~~

  Además elimina indentación de tabs en texto citado, colapsa líneas en blanco excesivas y limpia referencias cid: de
  imágenes embebidas.

  ## Tablas HTML en correos

  Cuando el cuerpo HTML de un correo contiene tablas, html2text normalmente las aplana. El módulo converters/html.py
  incluye _html_to_md_with_tables, que preprocesa las tablas HTML antes de convertir el resto del cuerpo, generando
  tablas Markdown válidas. Para .msg, el sistema procesa en paralelo el cuerpo de texto plano (para detectar separadores
   de hilo) y el cuerpo HTML (para preservar tablas), y los empareja segmento a segmento cuando los conteos coinciden.
 
  ## Frontmatter YAML: estructura lista para Obsidian

  Cada archivo de correo generado incluye un frontmatter YAML completo y consistente:

~~~yaml
  ---
  fecha: 2026-03-15
  de: "[[Alonzo Vera]]"
  para:
    - "[[Clara Cabrera]]"
    - "[[José Revollo]]"
  cc:
    - otro@empresa.com
  asunto: Revisión del contrato
  tipo: correo
  direccion: enviado
  tags:
    - correo
  adjuntos:
    - contrato_v2.pdf
  ---
~~~

  Los campos de, para y cc se resuelven automáticamente contra el sistema de aliases antes de escribirse.

  
  ## Reglas configurables: contact_aliases.json

  Este es uno de los puntos más potentes del sistema. En lugar de que los correos queden con cadenas de texto como
  "Alonzo Vera <alonzo.vera@empresa.com>" en los metadatos, el sistema las convierte a wikilinks de Obsidian: "[[Alonzo
  Vera]]".

  Las reglas se definen en un archivo JSON en la raíz del proyecto:

~~~json
  {
    "aliases": [
      {
        "alias": "\"[[Alonzo Vera]]\"",
        "match": ["Alonzo Vera", "Alonzo.Vera", "alvera"]
      },
      {
        "alias": "\"[[Clara Cabrera]]\"",
        "match": ["Clara Cabrera", "Clara.Cabrera"]
      }
    ]
  }
~~~

  Cada regla tiene:
  - match: lista de fragmentos a buscar (case-insensitive, busca subcadena en el campo completo)
  - alias: el valor de reemplazo, en formato wikilink para Obsidian

  El archivo se lee en caliente con cada conversión —no requiere reiniciar el servidor. Agregar un contacto nuevo al
  JSON tiene efecto inmediato en la próxima conversión.

  El sistema también detecta automáticamente si un correo fue enviado o recibido buscando los patrones del dueño del
  sistema (alonzo.vera, alvera) en el campo remitente, y escribe el campo direccion: enviado o direccion: recibido según
   corresponda.

  
  ## Arquitectura modular

  El proyecto está organizado para que cada formato sea independiente:

~~~
  converters/
  ├── docx.py       — convert_docx
  ├── pdf.py        — convert_pdf
  ├── html.py       — convert_html (+ helpers de tablas)
  ├── tabular.py    — convert_xlsx, convert_csv
  └── email/
      ├── thread.py   — lógica de separación de hilos
      ├── builders.py — construcción de frontmatter y Markdown
      ├── eml.py      — convert_eml
      └── msg.py      — convert_msg
~~~

  convert_to_md.py actúa como dispatcher: detecta la extensión del archivo y delega al conversor correcto. La UI Flask
  importa desde ese mismo dispatcher, por lo que CLI y web siempre usan exactamente la misma lógica de conversión.


##  Para qué sirve en la práctica

  - Archivo de correos corporativos: cada hilo de Outlook se convierte en archivos individuales con fecha real,
  remitente resuelto como wikilink, y adjuntos listados en frontmatter.
  - Ingestión de documentación: PDFs, Word y presentaciones de reuniones pasan directamente a la base de conocimiento.
  - Pipeline automatizado: el watcher convierte los .msg que Outlook arrastra a una carpeta de salida, sin intervención
  manual.
  - Búsqueda transversal: con todo en Markdown plano, cualquier herramienta (Obsidian, ripgrep, VS Code) puede buscar en
   correos, documentos y notas al mismo tiempo.

  ---
  Lo que queda configurable sin tocar código

| Qué | Dónde | Efecto |
|---|---|---|
| Aliases de contactos | contact_aliases.json | Resuelve nombres en frontmatter |
| Carpeta vigilada | UI web / POST /watch/start | Define qué carpeta monitorear |
| Carpeta de salida | Argumento -o en CLI | Destino de los .md generados |
| Patrones de ruido | _clean_msg_segment en thread.py | Qué líneas se eliminan del cuerpo |


  El proyecto está pensado como infraestructura personal: corre local, no necesita internet, no tiene base de datos, y
  produce archivos de texto plano que duran décadas. La inteligencia está en las reglas, y las reglas son editables.
  
Puedes acceder a su repo público en [GitHub](https://github.com/re-al-7/MD-Converter).