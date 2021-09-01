---
layout: post
title:  "Preparando datos para deeplearning con databricks"
date:   2021-07-21 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Databricks Python Dev AI Azure DataPlatform
---

En ocasiones necesitamos procesar toneladas de datos para entrenar un modelo nuevo. Lanzar la limpieza de datos en un entorno de trabajo de desarrollo puede no ser suficiente cuando se trabaja con mucha información y es por eso que databricks nos ayuda bastante en esta tarea.

Lo que vamos a hacer en este post es ver cómo limpiar nuestros datos que usaremos posteriormente para entrenar un modelo de red neuronal de tipo Bidirectional LSTM-Attention (algo que queda fuera del alcance de este post). Los datos que vamos a utilizar en este caso incluyen dominios de internet comprometidos por diversos tipos de malware y por supuesto un conjunto de datos como referencia positiva.

En este caso disponemos de 93 tipos de malware detectados, teniendo algunos de ellos comprometidos mas de 100M de elementos, por lo que puedes imaginarte que hablamos de bastantes datos y que nuestro laptop no va a poder en tiempo razonable procesarlo para poder prepararlos.

Como lo primero en todo proyecto de IA es preparar los datos, lo que vamos a hacer en este momento se centra en esa parte. Para ello resumo a alto nivel lo que vamos a hacer:

- Crear cluster Databricks
- Montar nuestro blob storage en nuestro cluster
- Subir los ficheros a procesar
- Procesar los ficheros para preparar los datos para el modelo
- Volcar los datos a un csv
- Destruir cluster Databricks (opcional)
- Destruir blob storage (opcional)


# Crear cluster Databricks

<mark> step-by-step deploy databricks </mark>

DBFS no permite subir ficheros de más de 2Gb de tamaño y puesto que nuestro conjunto de datos sobrepasa eso con creces, lo que vamos a hacer es montar un Azure blob storage en nuestro cluster y subir ahi nuestros datos con los que poder trabajar.

# Desplegar nuestro blob storage

<mark> step-by-step deploy blob storage </mark>

![crear contenedor](/img/posts_published_in_other_sites/procesar_datos_databricks/1.png)

# Subir los ficheros a procesar

Y ahora añadimos los ficheros .csv como block blob:

>Importante: Databricks por ahora solo permite montar ficheros subidos en modo **block blob**.

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

Ahora solo tienes que lanzarlo desde tu cluster:

![mount_blob](/img/posts_published_in_other_sites/procesar_datos_databricks/mount_blob_storage_1.png)

Y probar que efectivamente puedes acceder a tus ficheros:

![ls](/img/posts_published_in_other_sites/procesar_datos_databricks/ls.png)


# Preparar los datos

Yo soy bastante dato a utilizar [SQL](#scalasql), pero en este caso voy simplemente a utilizar databricks para preparar los datos rápido y luego volcar el resultado a un csv, que es lo que directamente utilizaré para entrenar mi modelo. Para ello lo que voy a hacer es un proceso en unas cuantas fases. Obviamente depende de cada escenario, tendrás que hacer una cosa u otra, pero para el escenario que necesito yo:

## Obtener un conjunto de datos base

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
  
  if (file_name == "majestic_million.csv"):
    df = spark.read.option("header",True).option("sep",",").csv(file_path).select("Domain")
    dga_class = "alexa"
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
output_parquet = "/tmp/output/people.parquet"
df_with_all_rows.write.mode("overwrite").parquet(output_parquet)

parqDF = spark.read.parquet(output_parquet)
parqDF.createOrReplaceTempView("prepared_top_level_domains")
```

## Chequeo de datos

Ahora vamos a chequear la distribución de datos para ver que no hay desmadres

```sql
%sql
-- test
select dga_class, count(*) from prepared_top_level_domains group by dga_class
```

![checksql](/img/posts_published_in_other_sites/procesar_datos_databricks/checksql.png)

### Duplicados

Vamos a ver cuántos duplicados tenemos:

```sql
%sql
-- cuantos duplicados?
with cte as(
select _c0,count(*)  from prepared_top_level_domains group by _c0 having count(*)>1
)
select count(*) from cte
```

![duplicados](/img/posts_published_in_other_sites/procesar_datos_databricks/duplicados.png)

Como vemos, tenemos un montón...que toca limpiar antes :)

### Limpiar duplicados

Para limpiar duplicados, vamos utilizar la estrategia de ROW_NUMBER:

```sql
%sql
-- clean duplicated domains
with cte as (
select _c0,dga_class,
row_number() over(partition by _c0 order by _c0  ) as rn
from prepared_top_level_domains )
select _c0 as domain,dga_class from cte where rn = 1
```

De esta forma nos vamos a quedar con únicamente los dominios que no están duplicados (y una muestra de los duplicados), de forma que ya tenemos nuestro dataset listo para ser exportado...salvo porque nos interesa aprovechar y barajarlo 

## Shuffeling de datos

Un requerimiento que vamos a tener a la hora de inyectar datos a la red neuronal que estamos montando va a ser que los datos estén barajados. Aprovechando que estamos en spark, vamos a hacer ese shuffeling con SQL y así nos ahorramos esa parte luego :)

```sql
%sql
select * from prepared_top_level_domains order by rand() limit 10
```

>NOTA: Shuffeling mediante ORDER BY RAND()

En este caso, realmente lo que vamos a barajar es el dataset sin los duplicados que hemos estado viendo antes, de esta forma la query quedará así:

```sql
%sql
-- clean duplicated domains
with cte as (
select _c0,dga_class,
row_number() over(partition by _c0 order by _c0  ) as rn
from prepared_top_level_domains )
select _c0 as domain,dga_class from cte where rn = 1 ORDER BY rand()
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

El resultado de ese output estará dentro del blob storage de nuestro azure, en una carpeta llamada "prepared_top_level_domains" y su contenido en un único fichero CSV con prefijo "part-"

![prepared](/img/posts_published_in_other_sites/procesar_datos_databricks/output.png)

Ya solo nos queda bajarnos el fichero y enchufárselo a nuestra red neuronal, pero eso será otra historia :)

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