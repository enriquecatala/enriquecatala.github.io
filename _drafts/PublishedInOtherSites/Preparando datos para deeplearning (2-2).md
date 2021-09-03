---
layout: post
title:  "Preparando datos para deeplearning con databricks (2/2)"
date:   2021-07-21 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Databricks Python Dev AI Azure DataPlatform
---

En la continuación de este post, vamos a exportar los datos del tokenizador. 

# Instalar tensorflow

Tendremos que instalar tensorflow 2.5 en este caso para poder hacer uso del tokenizador que posteriormente utilizaremos en nuestra red neuronal. 

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

```python
# save tokenizer
save_tokenizer_path = "/dbfs/mnt/sentinelml/tokenizer.json"

with open(save_tokenizer_path, 'w') as f:
  f.write(tokenizer.to_json())
```

![tokenizer](/img/posts_published_in_other_sites/procesar_datos_databricks/tokenizer.png)


## Salvar el array numérico

Al final aqui el problema que vamos a encontrarnos es que el conjunto de datos es suficientemente grande como para que no quepa en memoria RAM la llamada a [pad_sequences](https://www.tensorflow.org/api_docs/python/tf/keras/preprocessing/sequence/pad_sequences), que es <mark>monothread</mark> y nos dará un error de memoria 

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
```
```python
# convert the list to dataframe
# 49s
from pyspark.sql import Row
rdd1 = sc.parallelize(url_int_tokens)
row_rdd = rdd1.map(lambda x: Row(x))
f_url_int_tokens = sqlContext.createDataFrame(row_rdd)

```
```python
# apply the padding
x_padded = f_url_int_tokens.select("_1").withColumn("_1", pad_fix_length(f_url_int_tokens._1))
```

![padding](/img/posts_published_in_other_sites/procesar_datos_databricks/padding.png)

```python
```
```python
```
```python
```
```python
```
```python
```

```python
# To save array of ints into csv we need to serialize that array first
output_file_path = "/mnt/sentinelml/padded_int_domains" 
x_padded.withColumn("_1",F.concat_ws(",",F.col("_1"))).write.option("mode","overwrite").option("header","true").csv(path=output_file_path)
```