---
layout: post
title:  "Como evitar los temidos DeadLocks"
date:   2020-08-18 00:00:00 +0200
tipue_search_active: true
excerpt_separator: <!--end_excerpt-->
tags: SqlServer Blocking Performance
---

>NOTA: Este post ha sido migrado de mi blog oficial tal cual. Fue escrito en 2007

Lamentablemente no hay una respuesta única a este temido problema. Todo depende de muchos factores, especialmente de arquitectura y diseño del acceso a datos , pero puedo dar unos consejos que aunque pueden resultar obvios, estan muy relacionados con este tipo de problemas.

<!--end_excerpt-->

Transacciones, cuanto menos duren mejor
Quizas de lo mas importante a tener en cuenta es que las transacciones han de ser cuanto mas rápidas mejor. Si tienes que modificar un DataTable, hazlo en modo desconectado y fuera de la transacción, de forma que la transacción sea única y exclusivamente de modificacion de datos. Es muy comun ver en algunas empresas aperturas de transacciónes que duran minutos, pero que se pueden dejar en segundos si se deja el preprocesado de forma desconectada y luego se lanza un update completo de todo a la vez, sin calculos ni nada.

En definitiva, todo proceso que puedas sacar fuera de una transacción, sácalo.

## Lecturas Sucias

Si puedes utilizar el modo de aislamiento no bloqueante ( WITH (NOLOCK) ) al realizar consultas, utilizalo. Muchas veces es la propia aplicación la que de forma lógica impide que alguien modifique un dato de forma que no pasa nada si hacemos las consultas con NOLOCK. Esto esta muy relacionado con la arquitectura.

Un ejemplo de esto es por ejemplo la tipica aplicación de facturación que impide que dos usuarios modifiquen la misma factura evitando que la abra mas de uno a la vez. En este caso, todas las consultas las podremos hacer con lectura sucia porque sabemos que todo lo que obtendremos va a ser válido y no va a producirle error al cliente.

## Optimiza tus accesos a Base de Datos

Usa índices allí donde los necesitas. Un indice puede acelerar la operación notoriamente. Un update puede durar por ejemplo 1 minuto o puede durar 0.0000000001 segundos dependiendo del índice que utilicemos.
Al igual que un índice puede mejorar, el pasarse tambien puede empeorar ya que los índices no se mantienen solos y las inserciones se pueden ver afectadas.
El terreno de la optimización de las Bases de Datos es un terreno bastante peliagudo donde se combina el conocimiento de la aplicación ( qué datos se van a recuperar y a partir de qué otros ) con el de la BBDD a optimizar. Pese a esto, no es raro que este proceso sea infinito de forma que siempre estemos mejorándolo, ya que la BBDD suele ser para la mayoria un ente dinámico que avanza con el tiempo y la aplicación que soporta.


## Vigila tus herramientas

Si utilizas herramientas de terceros que accedan a datos ( herramientas de reporting , por poner un ejemplo ) , comprueba el tipo de aislamiento que esta utilizando dicha herramienta para realizar sus operaciones. En una ocasión me encontré que la mayoria de interbloqueos se debian a que una herramienta de reportes muy conocida estaba realizando las consultas con aislamiento SERIALIZABLE, y cuando un usuario intentaba imprimir un reporte bloqueaba a prácticamente todo el mundo. Esto la mayoria de ocasiones se soluciona en el RegEdit de Windows ;)

## Optimiza tu servidor de Base de Datos

Puede parecer algo secundario, pero la buena salud de un servidor afecta, y mucho, a la salud de la aplicación.
Todo lo anterior no nos sirve de mucho si nuestro servicio de SQL Server no dispone de memoria suficiente para realizar las operaciones y está continuamente paginando a disco. Encontrar una buena salud de nuestro servidor de BBDD tambien ha de estar en nuestra cabeza cuando queramos minimizar el impacto de una consulta contra la BBDD.

Como vemos, todo es muy genérico pero resulta importante tenerlo en mente siempre que diseñemos un acceso a datos , sea al motor de Base de Datos que sea.