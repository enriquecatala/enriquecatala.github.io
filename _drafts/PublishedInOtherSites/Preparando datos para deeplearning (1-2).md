---
layout: post
title:  "Preparando datos para deeplearning con databricks (1/2)"
date:   2021-07-21 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Databricks Python Dev AI Azure DataPlatform
---

Preparar datos para deeplearning, es una tarea muy dependiente de la entrada que necesite nuestra red y de los datos que tenemos. En este capítulo vamos a ver como preparar datos para deeplearning con databricks para un problema de clasificación aparentemente sencillo que es saber si un dominio de internet es malicioso o no simplemente con el texto del mismo. 

Vamos a disponer de un conjunto de datos muy grande de dominios de internet que tienen la catalogación de "buenos" y "malos", donde los "malos" son dominios generados por software malicioso. En principio no vamos a entrar en los detalles ahora del problema que queremos resolver ni del apartado relativo a la propia red neuronal, sino que vamos a verlo desde el punto de vista puro del tratamiento de datos. 

La red neuronal que vamos a entrenar va ser fundamentalmente una arquitectura de tipo LSTM con capa de atención (no vamos a entrar en sus detalles ahora). Puesto que su tarea va a ser "clasificación", lo que vamos a necesitar va a ser:
- Array de números con las características a utilizar para entrenar la red neuronal.
- Array de números con las etiquetas de cada dominio de internet.

Los datos que vamos a tener son un conjunto muy grande de dominios de internet y su categoria (junto a mas información irrelevante para este problema). 

Vamos a disponer por tanto de algo así:

| dominio | categoria |
|---------|-----------|
| google.com | clase1 |
| facebook.com | clase1 |
| tbitter.com | malo |
| gogle.com | malo |

Y lo que buscamos es tener algo así

| dominio_ids | categoria_ids |
|--------------|----------------|
| [0,0,..., 424, 234] | 0 |
| [0,0,...,,45, 35] | 1 |
...

Es decir, tendremos que convertir nuestros dominios a un array numérico y las etiquetas a números.

Como ves, si has trabajado con redes neuronales es un problema bastante sencillo, siendo aqui el único "problema" el volumen de datos que vamos a mover.

Respecto a los datos que vamos a utilizar: Disponemos de 93 tipos de malware detectados, teniendo algunos de ellos comprometidos mas de 100 millones de direcciones, por lo que puedes imaginarte que hablamos de bastantes datos y que nuestro laptop no va a poder en tiempo razonable procesarlo para poder prepararlos.

Para ello resumo a alto nivel lo que vamos a hacer:

- Crear cluster Databricks
- Montar nuestro blob storage en nuestro cluster
- Subir los ficheros a procesar
- Procesar los ficheros para preparar los datos para el modelo
   - Limpiar los datos
   - Calcular las clases
   - Tokenizar nuestros dominios y transformarlos de texto a array de números
- Volcar los datos a un csv
- Destruir cluster Databricks (opcional)
- Destruir blob storage (opcional)


# Crear cluster Databricks

Lo primero que vamos a hacer es crearnos nuestro cluster Databricks. Como es algo bastante sencillo y documentado, puede seguir esta [estupenda guia de Microsoft](https://docs.microsoft.com/en-us/azure/databricks/clusters/create)

En lo que a nosotros nos afecta, solo quiero añadir que puesto que **DBFS no permite subir ficheros de más de 2Gb de tamaño** y que nuestro conjunto de datos sobrepasa eso con creces, lo que vamos a hacer es montar un **Azure blob storage** en nuestro cluster y para subir ahi nuestros datos con los que poder trabajar.

# Desplegar nuestro blob storage

Tal como hemos comentado, vamos a necesitar un almacenamiento con el que poder trabajar, porque DBFS no nos va a dejar subir ficheros de más de 2Gb. Vamos en este caso a utilizar Azure Blob Storage por comodidad y precio. Para crear un blob storage te recomiendo que sigas esta [guia oficial](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-portal)

Una vez creado, añadiremos nuestro contenedor para trabajar con databricks, que en este caso le voy a llamar _sentinelml_
![crear contenedor](/img/posts_published_in_other_sites/procesar_datos_databricks/1.png)

# Subir los ficheros a procesar

Y ahora añadimos los ficheros .csv como <mark>block blob</mark>:

>Importante: Databricks por ahora solo permite trabajar con ficheros montados en modo **block blob**.

![block-blob](/img/posts_published_in_other_sites/procesar_datos_databricks/block-blob.png)

# Montar nuestro blob storage en nuestro cluster

Par hacerlo, podemos seguir este snippet

```scala
dbutils.fs.mount(
  source = "wasbs://<container-name>@<storage-account-name>.blob.core.windows.net",
  mount_point = "/mnt/<mount-name>",
  extra_configs = {"<conf-key>":dbutils.secrets.get(scope = "<scope-name>", key = "<key-name>")})
```

> NOTA: Para más información ver https://docs.databricks.com/data/data-sources/azure/azure-storage.html#mount-azure-blob

- \<storage-account-name> nombre del Azure Blob storage account (el que le pusieras)
- \<container-name> nombre del contenedor con los datos (el que le pusieras)
- \<mount-name> DBFS path donde montaremos el contenedor (puedes poner lo que quieras aqui, se montará con el nombre que le des)
- \<conf-key> Puede ser _fs.azure.account.key.\<storage-account-name>.blob.core.windows.net_ o _fs.azure.sas.\<container-name>.\<storage-account-name>.blob.core.windows.net_
  - Yo con esto me lié un poco :). Lo que quiere decir, es que pongas cualquiera de los 2 strings de texto tal cual aparecen con el replace correspondiente, luego verás un ejemplo
- _dbutils.secrets.get(scope = "\<scope-name>", key = "<\key-name>")_ obtiene la key de los secrets de databricks (opción preferida para producción)
  - En mi caso, puesto que va a ser para lanzar un procesado rápido, voy a ponerle la key1 de mi storage account directamente

De forma que un posible ejemplo podría ser:

```scala
dbutils.fs.mount(
  source = "wasbs://sentinelml@sentinelmlwstrololol.blob.core.windows.net",
  mount_point = "/mnt/sentinelml",
  extra_configs = {"fs.azure.account.key.sentinelmlwstrololol.blob.core.windows.net":"ziYC59ESQEF3zpTROLOLOL"})
```

Ahora solo tienes que lanzarlo desde tu cluster y ya quedará montado para siempre (mientras no lo desmontes):

![mount_blob](/img/posts_published_in_other_sites/procesar_datos_databricks/mount_blob_storage_1.png)

Y probar que efectivamente puedes acceder a tus ficheros:

![ls](/img/posts_published_in_other_sites/procesar_datos_databricks/ls.png)

> NOTA: Los ficheros los hemos subido en el punto anterior


# Preparar los datos

En este paso vamos a realizar varias cosas, algunas en pyspark y otras en sparksql. Aqui ya es a gusto del desarrollador, yo simplemente me siento mas cómodo trabajando con SQL y por eso verás mucha query :)

## Limpiar los datos

Lo primero va a ser obtener un conjunto de datos nivelado por clases. Puesto que tengo 2M de dominios catalogados como "buenos" y más de 100M catalogados como "malos", para minimizar el bias lo que voy a hacer es primero cargar cada clase con un conjunto de datos aproximadamente de 1M como mucho.


Al final, lo que ves aqui debajo es simplemente un procesado que va a leer todos los ficheros del blob storage y va a quedarse únicamente con el dominio y la clase a la que pertenece

```python
from pyspark.sql.functions import col,lit
hadoop = sc._jvm.org.apache.hadoop

fs = hadoop.fs.FileSystem
conf = hadoop.conf.Configuration()
path = hadoop.fs.Path('/mnt/sentinelml/')
first_iteration = True
number_of_files = len(fs.get(conf).listStatus(path))
i = 0
for f in fs.get(conf).listStatus(path):
  file_path = str(f.getPath())
  file_name = file_path.split('/')[-1]    
  
  # This one has the header
  if (file_name == "majestic_million.csv"):
    df = spark.read.option("header",True).option("sep",",").csv(file_path).select("Domain")
    dga_class = "alexa"
  # this one has the domain in the second column
  elif (file_name == "top-1m.csv"):
    df = spark.read.option("header",False).option("sep",",").csv(file_path).select("_c1")
    dga_class = "alexa"
  else:    
    # First columns is the only relevant (the domain)
    # since we have really big files, we only want 1M rows per class to avoid bias
    df = spark.read.option("header",False).option("sep",",").csv(file_path).select("_c0").limit(1000000)  
    dga_class = file_name.split('_')[0]    
    # rename column and adding the dga_class
    tbl= tbl.withColumnRenamed("_c0","domain")
    
  df= df.withColumn("dga_class",lit(dga_class))
  #df.show(2)
  if first_iteration:
    df_with_all_rows = df
    first_iteration=False
  else:
    df_with_all_rows = df_with_all_rows.union(df)
  #print(f'File: {file_name} - class: {dga_class} - rows: {df_with_all_rows.count()}')
  print(f'[{i}/{number_of_files}] File: {file_name} - rows: {df_with_all_rows.count()}')
  i+=1

print ("Process finished")  
```

![creating dataframe](/img/posts_published_in_other_sites/procesar_datos_databricks/creating_dataframe.png)

## Salvar a parquet

Para poder chequear rápido datos y ver que todo es correcto, vamos a salvar los datos a parquet y así poder validar que la cosa va bien:

```python
# Save to parquet
output_parquet = "/tmp/output/domains.parquet"
df_with_all_rows.write.mode("overwrite").parquet(output_parquet)

parqDF = spark.read.parquet(output_parquet)
# this is not required since we are going to load into database
parqDF.createOrReplaceTempView("prepared_top_level_domains")
```

### Meter los datos en base de datos (opcional)

En este caso finalmente me han salido unos ~20M de filas, que databricks se come en apenas 5s como parquet, por lo que no es necesario tampoco venirse arriba y cargar en BBDD...pero si quisieramos, podríamos hacer esto:

```sql
%sql
use sentinelml;

create table if not exists prepared_top_level_domains(_c0 STRING comment "this is the domain name",
                                              dga_class STRING comment "this is the class of the domain")
using parquet
options(
path "/tmp/output/domains.parquet"
)
--partitioned by (dga_class)
--clustered by (_c0) into 4 buckets
```

Con lo que ahora tendríamos accesible tambien los datos a nivel fijo en la BBDD llamada "sentinelml" que me cree yo previamente.

>NOTA: La diferencia entre hacer esto o no, es que en los siguientes scripts tendrias que añadir "sentinelml." justo delante del nombre de la vista. Si hicieste un createOrReplaceTempView, no haría falta ponerle la referencia a BBDD, pero estará ejecutando otro motor diferente, que a penas vas a notar para los pocos datos que hay.

## Chequeo de datos

Ahora vamos a chequear la distribución de datos para ver que no hay desmadres

```sql
%sql
-- test
select dga_class, count(*) 
from prepared_top_level_domains 
group by dga_class
```

![checksql](/img/posts_published_in_other_sites/procesar_datos_databricks/checksql.png)

### Duplicados

Vamos a ver cuántos duplicados tenemos:

```sql
%sql
-- cuantos duplicados?
with cte as(
  select _c0,count(*)  
  from prepared_top_level_domains 
  group by _c0 
  having count(*)>1
)
select count(*) from cte
```

![duplicados](/img/posts_published_in_other_sites/procesar_datos_databricks/duplicados.png)

Como vemos, tenemos un montón...que toca limpiar antes :)

### Limpiar duplicados

Para limpiar duplicados, vamos utilizar la estrategia de [ROW_NUMBER](https://docs.databricks.com/sql/language-manual/functions/row_number.html):

```sql
%sql
-- clean duplicated domains
with cte as (
  select _c0,
         dga_class,
         row_number() over(partition by _c0 order by _c0  ) as rn
  from prepared_top_level_domains 
  )
select _c0 as domain,dga_class from cte where rn = 1
```

De esta forma nos vamos a quedar con únicamente los dominios que no están duplicados (y una muestra de los duplicados), de forma que ya tenemos nuestro dataset listo para ser exportado...salvo porque nos interesa aprovechar y barajarlo 

## Shuffeling de datos

Un requerimiento que vamos a tener a la hora de inyectar datos a la red neuronal que estamos montando va a ser que **los datos estén barajados**. Aprovechando que estamos en spark, vamos a hacer ese shuffeling con SQL y así nos ahorramos esa parte luego :)

```sql
%sql
-- para que veas el ejemplito
select * 
from prepared_top_level_domains 
order by rand() 
limit 10
```

>NOTA: Shuffeling de dga_class mediante ORDER BY [RAND()](https://docs.databricks.com/sql/language-manual/functions/rand.html)

![shuffeling](/img/posts_published_in_other_sites/procesar_datos_databricks/shuffeling.png)

En este caso, realmente lo que vamos a barajar es el dataset sin los duplicados que hemos estado viendo antes, de esta forma la query quedará así:

```sql
%sql
-- clean duplicated domains
with cte as (
  select _c0,dga_class,
  row_number() over(partition by _c0 order by _c0  ) as rn
  from prepared_top_level_domains 
)
select _c0 as domain,dga_class 
from cte 
where rn = 1 
ORDER BY rand()
``` 

## Salvar a CSV

Ya por último solo nos falta salvar el resultado a un fichero CSV. En mi caso despues de revisar tamaños y demás, he visto que me es mas cómodo tenerlo todo en un único CSV, asi que lo que vamos a hacer es lanzar la query y salvar el resultado directamente a un CSV utilizando un único worker, por eso verás un "coalesce(1)"

```python
output_file_path = "/mnt/sentinelml/prepared_top_level_domains" 
sql_text = f"""
with cte as (
  select _c0,dga_class,
  row_number() over(partition by _c0 order by _c0  ) as rn
  from prepared_top_level_domains 
)
select _c0 as domain,dga_class from cte where rn = 1 ORDER BY rand()"""
spark.sql(sql_text).coalesce(1).write.option("mode","overwrite").option("header","true").csv(path=output_file_path)
```

El resultado de ese output estará dentro del blob storage de nuestro azure, en una carpeta llamada "prepared_top_level_domains" y su contenido, gracias a haber forzado "_coalesce(1)_" en **un único fichero CSV** con prefijo "part-" que es el que más tarde me llevaré para el entrenamiento de mi red

![prepared](/img/posts_published_in_other_sites/procesar_datos_databricks/output.png)

Este dataset es el que contiene todos los dominios y sus clases, pero en modo texto, por lo que no podemos utilizarlo para entrenar nuestra red neuronal directamente, sino que tendremos que convertirlos a array de números, pero eso lo haremos en la siguiente parte de este post.
## Cuántas clases tenemos

Como siguiente tarea, ahora nos queda saber cuantas clases tenemos, para que nuestro clasificador lo sepa y asignarles sus id a nombres correspondientes.

```sql
select distinct row_number() over (order by dga_class) dga_class_id,
       dga_class
from prepared_top_level_domains group by dga_class
```

![clases](/img/posts_published_in_other_sites/procesar_datos_databricks/classes.png)


Y listo!, con esto ya tenemos las 2 cosas mínimas necesarias para comenzar a plantear las ultimas fases de preparación de datos para nuestra red

En el siguiente post, veremos cómo preparar el tokenizador y la conversión de estos dominios al array numérico que necesitamos. 

# Apéndice

## SCALASQL

La otra opción que podemos utilizar es cargar los datos directamente y procesar mediante SQL

```sql
%sql
create table if not exists sentinelml.banjori(domain STRING comment "this is the domain name",
                                                domain_generator_internal_id int comment "this is not relevant info",
                                                generator_name_instance_id STRING comment "this is the generator id that generated the row")
using csv
options(
path '/mnt/sentinelml/banjori_dga.csv',
sep ',',
header false
)
-- y aqui todas y cada una de nuestras tablas
-- ...
```

Y luego montar una query que devuelva lo mismo de antes

```sql
%sql 
select domain, generator_name
from (select domain, 
       generator_name_instance_id, 
       substring(generator_name_instance_id,0,instr(generator_name_instance_id,'_')-1) generator_name
from sentinelml.banjori
LIMIT 1000000)
union 
select domain,        
       substring(generator_name_instance_id,0,instr(generator_name_instance_id,'_')-1) generator_name
from sentinelml.randomloader
```

El problema de hacerlo así es que como es para algo rápido y que tenemos un conjunto de ficheros variable...no es muy útil hacerlo así. 

>nota: en este caso no necesitamos hacer nada más con la información, por lo que tampoco sería necesario cargarlo a parquet para hacer nada con ello