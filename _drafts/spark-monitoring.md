---
layout: post
title:  "Monitorizacion databricks con Log-Analytics"
date:   2021-11-19 00:00:00 +0200
tipue_search_active: true
excerpt_separator: <!--end_excerpt-->
tags: Databricks Azure DataPlatform Dev Performance
---

# Monitorizacion databricks con Log-Analytics

Utilizando el respositorio oficial de Microsoft Patterns&Practices encontramos el proyecto [Spark monitoring](https://github.com/mspnp/spark-monitoring) que nos permite monitorizar el estado de nuestros clusters Spark para inyectar en un Azure Log Analytics tanto métricas de ganglia, como logs de [log4j](https://logging.apache.org/log4j/2.x/manual/configuration.html).

La forma de trabajar a grandes rasgos será la siguiente:
1) Descargamos el proyecto
2) Seleccionamos la versión de Databricks y spark que queremos monitorizar
3) Compilamos el proyecto para la versión de Spark a monitorizar
4) Desplegamos la libreria compilada al cluster spark
5) Configurar la libreria en el cluster
6) Introducir su ejecución en el cluster como init-script


# Descargar el proyecto

Para descargar el proyecto:

```bash
git clone https://github.com/enriquecatala/spark-monitoring.git spark-monitoring
```
>NOTA: En mi repositorio de GitHub, realicé un par de moficiaciones que en el momento de realizar todavía no estaban incluidas en el proyecto oficial. Podeis probarlo con el repositorio oficial y si todo va OK, será mejor

# Seleccionar la versión de Databricks y spark que queremos monitorizar

Es importante conocer la versión del cluster databricks contra el que vayamos a realizar monitorización porque la versión a desplegar nos va a delimitar la versión para la que compilar la libreria. Necesitaremos la tupla de versiones "spark-scala", que se puede obtener mirando en la sección de clusters de databricks:

![clusters](/img/posts_published_in_other_sites/spark-monitoring/cluster_versions.png)

## databricks-cli

Mediante [databricks-cli](https://docs.databricks.com/dev-tools/cli/index.html), se puede obtener la lista igualmente de esta forma:

```bash
# get list of clusters and versions
databricks clusters list --output json | jq '.clusters[] | "Cluster:" + .cluster_name + " -- Version: " + .spark_version'
```

Output:
```bash
"Cluster:cluster1 -- Version: 9.0.x-scala2.12"
"Cluster:cluster2 -- Version: 9.1.x-scala2.12"
"Cluster:cluster3 -- Version: 9.0.x-scala2.12"
```

Ahora para cada version tendremos que ir a ver su par de versiones de spark y de databricks.
```bash
# get the spark-scala pair for the databricks version named: 9.0.x-scala2.12
databricks clusters spark-versions | jq '.versions[] | select(.key=="9.0.x-scala2.12")' | grep name 
```

Output:`"name": "9.0 (includes Apache Spark 3.1.2, Scala 2.12)"`

# Compilamos el proyecto para la versión de Spark a monitorizar

Una vez tenemos ya la versión del cluster que vamos a monitorizar, podemos compilar el proyecto para la versión de Spark que queremos monitorizar. En el ejemplo que voy a hacer a continuación lo voy a compilar para la versión de scala-2.12 y spark-3.1.2

Lo único que debemos modificar de la siguiente ejecución es el contenido del parámetro -P, que en el caso del ejemplo es `"scala-2.12_spark-3.1.2"` pero en tu caso dependerá de la versión que hayas seleccionado:

```bash
cd spark-monitoring/
chmod +x ./build.sh
# To build a single profile (latest long term support version):
docker run -it --rm -v `pwd`:/spark-monitoring -v "$HOME/.m2":/root/.m2 -w /spark-monitoring/src maven:3.6.3-jdk-8 mvn install -P "scala-2.12_spark-3.1.2"
```
>NOTA: Esto levantará un docker con la configuracion java y requisitos necesarios para compilar el proyecto, dejando la libreria compilada en la carpeta **_spark-monitoring/src/target_** 

# Desplegamos la libreria compilada al cluster spark

Ahora copiaremos la libreria compilada al cluster spark. Para ello, podemos utilizar el script que hice llamado "upload_with_dbfs.sh" 

```bash
# Upload library to dbfs
sh spark-monitoring/upload_with_dbfs.sh 3.1.2_2.12-1.0.0
```

>NOTA: Presta atención a qué cluster estás referenciando porque el script va a usar el cluster que esté configurado para [databricks-cli](https://docs.databricks.com/dev-tools/cli/index.html).

## Checkeo

Para ver que efectivamente está subida la libreria al cluster, deberias de poder lanzar `dbfs ls dbfs:/databricksecb/spark-monitoring` y ver que se han subido estos 3 ficheros:
```bash
$ dbfs ls dbfs:/databricksecb/spark-monitoring
spark-listeners-loganalytics_3.1.2_2.12-1.0.0.jar
spark-listeners_3.1.2_2.12-1.0.0.jar
spark-monitoring.sh
```
>NOTA: Requiere [databricks-cli](https://docs.databricks.com/dev-tools/cli/index.html)

# Configurar la libreria en el cluster

Dado que los datos de monitorización van a subirse a Azure Log Analytics, necesitaremos configurar la libreria para que pueda subirse a ese repositorio. Para ello, necesitaremos una cuenta de Azure Log Analytics, un workspace y un proyecto. 

Una vez tengamos eso, necesitaremos los siguientes valores:
- workspace id
- (primary|secondary) key 

Que podemos ver en la siguiente imagen cómo obtener:

![loganalytics](/img/posts_published_in_other_sites/spark-monitoring/loganalytics.png)


## Crear azure log analytics workspace

En caso de que no se disponga un Azure Log Analytics, Microsoft provee de un template ARM que nos permite crear un workspace con las consultas Kusto que vamos a necesitar posteriormente.

Podemos ver el [template ARM aqui](https://github.com/mspnp/spark-monitoring/blob/master/perftools/deployment/loganalytics/logAnalyticsDeploy.json)

# Introducir los valores en un keyvault

Para mas seguridad, se recomienda obtener los valores de un keyvault en lugar de introducirlos directamente en los parámetros de inicio de la libreria. Para ello, necesitaremos una cuenta de Azure KeyVault, donde iremos dando de alta como secrets los valores anteriormente obtenidos.

![lak](/img/posts_published_in_other_sites/spark-monitoring/loganalytics-key.png)

Una vez tenemos los valores introducidos en nuestro keyvault ([ya vinculado a nuestro databricks](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes)) iremos al cluster environment variables y pegaremos las siguientes líneas:
```bash
LOG_ANALYTICS_WORKSPACE_ID={{secrets/keyvault-ecb-scope/databricksloganalytics-workspace-id}}
LOG_ANALYTICS_WORKSPACE_KEY={{secrets/keyvault-ecb-scope/databricksloganalytics-key}}
```
>NOTA: keyvault-ecb-name es el nombre del [keyvault registrado en databricks que tengas](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes), que en mi cluster es keyvault-ecb-scope.

De esta forma, nuestras environment variables podrían quedar de esta forma:

![law](/img/posts_published_in_other_sites/spark-monitoring/loganalyticsworspace.png)


## Validar nuestro secret scope y variables

Para validar que todo funcione correctamente, podemos ver qué secret scope/key tenemos accesibles desde nuestro cluster:

```bash
❯ databricks secrets list-scopes
Scope               Backend         KeyVault URL
------------------  --------------  -------------------------------------
keyvault-ecb-scope  AZURE_KEYVAULT  https://mykeyvaultrandomxx22.vault.azure.net/

❯ databricks secrets list --scope keyvault-ecb-scope                                                 
Key name                               Last updated
-----------------------------------  --------------
databricksloganalytics-key            1635246893000
databricksloganalytics-workspace-id   1635246863000

```

# Introducir su ejecución en el cluster como init-script

Finalmente ya es hora de introducir en nuestro init script el arranque de la libreria. Para ello haremos referencia al fichero anteriormente desplegado llamado "spark-monitoring.sh"

![is](/img/posts_published_in_other_sites/spark-monitoring/initscripts.png)

### Testeo de arranque

Hora de arrancar el cluster e ir a la sección "Event Log" para validar que efectivamente el init script se ha levantado correctamente

![initok](/img/posts_published_in_other_sites/spark-monitoring/init-ok.png)

# Testeo de monitorización

## Logs from log4j

Para probar que funcione correctamente, abre cualquier notebook e introduce el siguiente código pyspark:

```python
spark_log4j = sc._jvm.org.apache.log4j
logger = spark_log4j.LogManager.getLogger(__name__)

logger.info("Test log message")
logger.warn("Test warn message")
logger.error("test error message")
```

Ahora ve a tu log analytics y lanza la siguiente consulta kusto:

```kusto
SparkLoggingEvent_CL
| project TimeGenerated, clusterId_s, Message, Level, logger_name_s, clusterName_s
| sort by TimeGenerated desc
| limit 1000
```

Si todo ha ido bien, deberias ver algo parecido a esto:

![kusto1](/img/posts_published_in_other_sites/spark-monitoring/kustoquery1.png)

## Metrics

Para testear que se están subiendo datos de metricas, prueba algunas consultas como estas:

### DRIVER

```kusto
SparkMetric_CL 
| project TimeGenerated, name_s, value_d, count_d, values_s, metric_type_s, executorId_s, clusterName_s, mean_d, max_d, min_d, p95_d, p99_d
| where name_s  contains "driver"
| limit 1000
```

### CPU

```kusto
SparkMetric_CL
| where name_s contains "executor.cpuTime"
| extend sname = split(name_s, ".")
| extend executor=strcat(sname[0], ".", sname[1])
| project TimeGenerated, cpuTime=count_d / 100000
```

