---
layout: post
title:  "Preparando datos para deeplearning con databricks (2/2)"
date:   2021-07-21 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Databricks Python Dev AI Azure DataPlatform
---

Si recuerdas en el post anterior, lo que hicimos fue la tarea bruta de tomar todos los datos y procesarlos para dejarlos limpios y preparados para comenzar la última parte de preparación necesaria para el entrenamiento.

En mi caso, esta información la voy a utilizar como entrada en una red neuronal recurrente de tipo LSTM con capa de atención. Este tipo de redes requieren de una entrada de datos muy concreta, que por no extenderme mucho, necesita que esté fija en tamaño y que sea de forma numérica. Es decir, que ahora lo que tendremos que hacer es convertir esos 22M de dominios en una matriz de 22M x n, siendo n el número de columnas de mi array con el que codifiquemos los dominios...que aunque todavia no lo sabemos, deberia ser una cantidad igual al máximo numero de carácteres permitidos por un dominio. 

# Instalar tensorflow

Lo primero que tendremos que hacer ahora es ya instalar tensorflow 2.5 , porque es lo que vamos a utilizar para tokenizar nuestros datos. 

Desde una celda nueva en nuestro notebook de databricks, ejecutamos el siguiente comando: 

```bash
!pip install tensorflow==2.5
```

Una vez instalado tensorflow, ya podremos hacer uso de él. En este caso una de las tareas que mas nos van a tardar es la de generar el procesado de los tokens que queremos que reconozca nuestra red neuronal. En este caso como tenemos el texto de los dominios, lo que vamos a hacer es que cada letra sea un token, por lo que el modo de proceder será:

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
from tensorflow.keras import models
from tensorflow.keras.layers import LSTM, Dropout, Dense, Embedding
from tensorflow.keras.models import Sequential, model_from_json
from tensorflow.keras.callbacks import ModelCheckpoint
from tensorflow.keras.preprocessing.text import Tokenizer, tokenizer_from_json
from tensorflow.keras.preprocessing import sequence
from tensorflow.python.keras.engine import input_layer

# tokenizador con char_level=True
tokenizer = Tokenizer(filters='', lower=True, char_level=True)
```

>NOTA: Nuestro tokenizador va a tratar cada caracter como un token (no es lo habitual) y ademas vamos a presumir que el texto es en minusculas. 
Esto último es particularmente importante recordarlo a la hora de hacer la inferencia :)

# Cargar los dominios

Si sigues el post anterior el objeto lo tendrás ya preparado, pero si no es tu caso, igual tienes que volver a cargarlo. 

```python
output_parquet = "/tmp/output/domains.parquet"
parqDF = spark.read.parquet(output_parquet)
```

# Convertir los dominios a una lista

Para poder entrenar nuestra red neuronal, tenemos que convertir nuestros elementos a un array numérico, para lo que vamos a utilizar el propio keras...pero por desgracia tenemos que enviarle una lista de strings, lo que nos fuerza a convertir nuestros datos a ese formato. 

Esto lo podemos hacer con el siguiente comando pyspark:

```python
# ~27s
domains = parqDF.select("_c0").rdd.flatMap(lambda x: x).collect()
```

>NOTA: En este momento, nuestro objeto _"domains"_ es un list[str]

# Rellenar nuestro vocabulario de tokenizacion

Una vez ya tenemos nuestro dominio de texto en una lista, lo que tendremos que hacer es actualizar el vocabulario interno de nuestro tokenizador con esa información:

```python
# ~3mins
tokenizer.fit_on_texts(domains)
```

>NOTA: Para más información [fit_on_texts](https://www.tensorflow.org/api_docs/python/tf/keras/preprocessing/text/Tokenizer#fit_on_texts)


# Convertir nuestros dominios a array numérico

Justo lo que necesitamos ahora...convertir nuestros dominios a un array numérico ya es posible porque el tokenizador ya es capaz de hacer las conversiones puesto que tine el vocabulario necesario

Para hacerlo, lo que nos queda por tanto es:

```python
#5m
url_int_tokens = tokenizer.texts_to_sequences(domains)
```

>NOTA: En este momento _url_int_tokens_ va a ser un objeto de tipo list[list[int]]

![url_int_tokens](/img/posts_published_in_other_sites/procesar_datos_databricks/url_int_tokens.png)

# Comprobación

Tenemos ~23M de dominios, seguro que hemos sido capaces de encontrar entre ellos, dominios que tengan [longitud máxima de 253](https://en.wikipedia.org/wiki/Domain_name), que es el máximo posible para un dominio de internet. 
Vamos a comprobarlo:

```python
max_len = len(max(url_int_tokens, key=len))
```

![max_len](/img/posts_published_in_other_sites/procesar_datos_databricks/max_len.png)

# Salvar el tokenizador

Es importantísimo salvar el tokenizador, para que todo el progreso que hemos conseguido sirva para algo :). De esta forma nos ahorraremos todo esto en la parte del entrenamiento de nuestra red neuronal (que se sale del foco de este artículo):

```python
# save tokenizer
save_tokenizer_path = "/dbfs/mnt/sentinelml/tokenizer.json"

with open(save_tokenizer_path, 'w') as f:
  f.write(tokenizer.to_json())
```

Si recordais del anterior post, nosotros conectamos Databricks a nuestro azure blob storage para poder trabajar mas cómodamente. De esta forma, si nos vamos al contenedor donde le hemos dicho, veremos ahi el tokenizador, que será buena idea salvar :)

![tokenizer](/img/posts_published_in_other_sites/procesar_datos_databricks/tokenizer.png)


## Salvar el array numérico

Por último ya lo que nos queda es aprovecharnos de Databricks para dejarnos ya mascadito al máximo la información. Realmente en la máquina que voy a hacer yo el entrenamiento no tengo problema de memoria (+120Gb de RAM) por lo que no seria necesario este paso, pero lo normal no va a ser eso y por tanto te vendrá bien tambien ser capaz de salvar el array numérico que contiene las transformaciones de nuestro tokenizador de texto a número.

Concretamente, nos vamos a encontrar este problema debido a la función [pad_sequences](https://www.tensorflow.org/api_docs/python/tf/keras/preprocessing/sequence/pad_sequences), que es <mark>monothread</mark> y nos dará un error de memoria. Esta función se encarga de conseguir una de las cosas necesarias para el entrenamiento de un LSTM, que es que TODAS LAS MUESTRAS DEBEN TENER EL MISMO NUMERO DE CARACTERISTICAS (columnas en nuestro caso que se corresponden con letras del dominio).

```python
# OUT OF MEMORY!
x = sequence.pad_sequences(url_int_tokens, maxlen=max_len)
```

Como solución, lo que yo he hecho ha sido hacerme la función pad_sequence a mano, que al final es una cosa bastante sencilla, porque lo que hacemos es simplemente rellenar por ceros delante o detrás aquellos elementos que no lleguen a tener los 253 valores.

```python
# Workaround to fill of 0
from pyspark.sql.types import ArrayType, IntegerType
import pyspark.sql.functions as F

pad_fix_length = F.udf(
    lambda arr: arr[:253] + [0] * (253 - len(arr[:253])), 
    ArrayType(IntegerType())
)

# convert the list to dataframe
# 49s
from pyspark.sql import Row
rdd1 = sc.parallelize(url_int_tokens)
row_rdd = rdd1.map(lambda x: Row(x))
f_url_int_tokens = sqlContext.createDataFrame(row_rdd)


# apply the padding
x_padded = f_url_int_tokens.select("_1").withColumn("_1", pad_fix_length(f_url_int_tokens._1))
```

Aqui puedes ver un poco el resultado:

![padding](/img/posts_published_in_other_sites/procesar_datos_databricks/padding.png)

# salvar el array numerico

Ahora el último problema a salvar es que nuestro objeto DataFrame es un array de arrays, lo que se complica a la hora de ser almacenado en CSV.

La solución parece sencilla pero he estado bastante tiempo para encontrarla. Consiste básicamente en exportar como string el contenido de la columna "_1" (que es la que contiene los valores en mi caso, porque no le he puesto ningún nombre decente :)

```python
# To save array of ints into csv we need to serialize that array first
output_file_path = "/mnt/sentinelml/padded_int_domains" 
x_padded.withColumn("_1",F.concat_ws(",",F.col("_1"))).write.option("mode","overwrite").option("header","true").csv(path=output_file_path)
```

Y ya lo tendríamos, nuestra misma información generada en el post1, pero esta vez ya lista y preparada para ser consumida por la red neuronal!

![final](/img/posts_published_in_other_sites/procesar_datos_databricks/final.png)

![finalcsv](/img/posts_published_in_other_sites/procesar_datos_databricks/final_csv.png)