---
title: "Generar colores hexadecimales aleatorios con Postgres"
date: 2014-11-13
tags: ["sql"]
---

Hoy me he visto en la necesidad de obtener Varias fechas de una tabla en PostgreSql y mostrar cada una con un color diferente en un Sistema Web.

![_config.yml](/images/Colores-hexadecimales-con-PostgreSql.png)

Mi primera alternativa era buscar una funcion en PHP que me permitiera generar colores hexadecimales aleatorios, pero como tengo más experiencia en PLPgSql, decidí empezar por ahí. 

Primero lo primero: todos sabemos que un color hexadecimal es la presentacion de 3 numeros en el rango de 0-255 convertidos a base 16. Pues bien por ahí es por donde empece: generar un numero aleatorio en el rango de 0 a 255. 

~~~sql
SELECT trunc(random() * 255)::INTEGER
~~~

Ahora necesitamos una función que convierta un numero de Base 10 a Base 16: Para ello necesitamos la siguiente funcion: 

~~~sql
CREATE OR REPLACE FUNCTION b10_b16(digits bigint, min_width integer DEFAULT 0)
  RETURNS character varying AS
$BODY$
DECLARE
    chars char[]; 
    ret varchar; 
    val bigint; 
BEGIN
    chars:=ARRAY['0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'];
    val := digits; 
    ret := ''; 
    IF val < 0 THEN 
        val := val * -1; 
    END IF; 
    WHILE val != 0 LOOP 
        ret := chars[(val % 16)+1] || ret; 
        val := val / 16; 
    END LOOP;

    IF min_width > 0 AND char_length(ret) < min_width THEN 
        ret := lpad(ret, min_width, '0'); 
    END IF;

    RETURN ret;
END;
$BODY$
  LANGUAGE plpgsql IMMUTABLE
  COST 100;
~~~

El uso de ésta función es: 

~~~sql
SELECT b10_b16(45);
--Devolverá el Dato 2D

SELECT b10_b16(11);
--Devolverá el Dato B

SELECT b10_b16(11,2)
--Devolverá el Dato 0B
~~~

Con todo ésto tenemos la Base para generar un color aleatorio: 

~~~sql
SELECT '#' || 
        b10_b16(trunc(random() * 255)::INTEGER,2) || 
        b10_b16(trunc(random() * 255)::INTEGER,2) || 
        b10_b16(trunc(random() * 255)::INTEGER,2) AS color

--Devolvera algo así: #69A51D
~~~

Y listo!!!

Ya tenemos nuestra función que genera colores hexadecimales ALEATORIAMENTE!!! 