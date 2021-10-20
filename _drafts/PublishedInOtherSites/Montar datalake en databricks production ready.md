---
layout: post
title:  "Montar datalake en Databricks production ready"
date:   2021-07-21 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Databricks Python Dev Azure DataPlatform
---

En este post vamos a ver cómo montar nuestro [Azure DataLake Storage Gen2](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction) **production ready** utilizando [Databricks Secret Scopes](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes) y [Azure Keyvault](https://azure.microsoft.com/en-us/services/key-vault/)

Vamos a partir de esta situación de entrada:

![1](/img/posts_published_in_other_sites/montar_datalake_databricks/1.png)
>NOTA: Asumiremos que tenemos ya creado nuestro Azure Databricks workspace, nuestro Storage Account y un Azure KeyVault. 

<!--end_excerpt-->

![1](/img/posts_published_in_other_sites/montar_datalake_databricks/storageaccount.png)


# Crear service app y secret

Para poder vincular nuestro datalake a databricks, necesitamos generar un token de seguridad, asignarle permisos y luego usarlos desde nuestro Databricks workspace. Lo primero por tanto es generar un service app y un secret:

![1](/img/posts_published_in_other_sites/montar_datalake_databricks/create_serviceapp_and_secret.png)

>IMPORTANTE: En el último paso hemos creado un secret. Recuerda apuntar en algun sitio de forma temporal el "Value" porque no podras acceder a él nunca más y lo vamos a necesitar en el futuro para guardarlo en Azure KeyVault.

# Asignar permisos en datalake a nuestro service app 

Ahora que ya tenemos el service app creado y nuestro secret, vamos a asignarle permisos a nuestro datalake. Para ello, vamos a la sección de "Storage account" en la pestaña de "Access Control" y seleccionamos "Add role assigmnent". Lo que tendremos que hacer es asignar a nuestro service app el permiso de "Storage Blob Data Contributor".

![role assignment](/img/posts_published_in_other_sites/montar_datalake_databricks/role_assignment.png)

>NOTA: El service app, ahora puede acceder a todo el datalake, nos falta poder entonces usarlo desde databricks

# Databricks secret scope

Un databricks secret scope es un almacen que sirve para **guardar de forma segura datos críticos** y poder **referenciarlos desde nuestros notebooks sin conocer sus valores**. La idea será almacenar finalmente en un recurso de seguridad externo (como Azure KeyVault) nuestras credenciales (la que acabamos de crear), y luego poder referenciarlas desde nuestros notebooks de forma segura para que nadie pueda verlas y de esta forma acceder desde fuera a nuestro datalake.

Lo primero por tanto será disponer de nuestro Azure KeyVault.

# Crear KeyVault

La forma correcta de almacenar nuestros secrets va a ser utilizar Azure Keyvault, porque así lo tenemos desde fuera del cluster Databricks. Lo primero que haremos es crearnos un Azure KeyVault para almacenar nuestras credenciales de nuestro Storage Account. Si no tienes uno, crealo. Vamos a asumir que lo tenemos ya creado.

Una vez tengamos el keyvault vamos a linkarlo a nuestro databricks secret scope.

## Crear los secrets que vamos a necesitar

Vamos a necesitar los siguientes valores para montar nuestro datalake desde Databricks:
- _databricks-app-client-id_: Es el client id de nuestro service app
- _databricks-app-tenant-id_: Es el tenant id de nuestro service app
- _databricks-app-client-secret_: Es el valor que hemos generado para nuestro secret en el service app 

Por tanto, vamos a ir a nuestro KeyVault y los vamos a añadir:

![keyvault_secrets](/img/posts_published_in_other_sites/montar_datalake_databricks/keyvault_secrets.png)

# Generar nuestro Databricks secret scope

Ahora, vamos a aprovechar que tenemos los datos necesarios ya listos en KeyVault, para generar nuestro secret scope y permitir que haga uso de ellos. Lo primero será entrar en la pantalla de _#secrets/createScope_ de nuestro portal databricks.

Para poder entrar, ves a la url del dashboard de tu databricks, que puedes encontrar aqui:

![databricks-dashboard](/img/posts_published_in_other_sites/montar_datalake_databricks/databricks-dashboard.png)

y le añades al final la coletilla #secrets/createScope

por ejemplo: https://adb-ID.ID2.azuredatabricks.net#secrets/createScope

Una vez dentro, necesitarás de tu KeyVault los valores de DNS y ResourceID, que puedes encontrar aqui:

![keyvault_properties](/img/posts_published_in_other_sites/montar_datalake_databricks/keyvault_properties.png)

Y los introduces

![secret_scope](/img/posts_published_in_other_sites/montar_datalake_databricks/secret_scope.png) 

>NOTA: Hay una versión de [databricks cli](https://docs.databricks.com/dev-tools/cli/secrets-cli.html) que permite hacer todo esto programáticamente

## Probar secret scope

Vamos a probar que funciona nuestro secret scope. Para ello, abrimos ya un notebook de databricks...

```python
dbutils.secrets.list("keyvault-ecb-scope")
dbutils.secrets.get(scope = "keyvault-ecb-scope", key = "databricks-app-client-id")
```

![dbutils_get_secret](/img/posts_published_in_other_sites/montar_datalake_databricks/dbutils_get_secret.png)

>NOTE: En este cluster de ejemplo he deshabilitado la opción de credential Passthrough del cluster por testear que funcione el secret scope correctamente para forzar que no utilice mis credenciales para passthrough

![disable_credential_passthrough_testing_purposes](/img/posts_published_in_other_sites/montar_datalake_databricks/disable_credential_passthrough_testing_purposes.png)

# Montar datalake en databricks

Ya por fin, despues de todo esto podemos ir directamente a montar nuestro volumen:

```python
# get values from secret scope (linked to Azure Keyvault)
storage_account_name = "databricksecb"
client_id            = dbutils.secrets.get(scope="keyvault-ecb-scope", key="databricks-app-client-id")
tenant_id            = dbutils.secrets.get(scope="keyvault-ecb-scope", key="databricks-app-tenant-id")
client_secret        = dbutils.secrets.get(scope="keyvault-ecb-scope", key="databricks-app-client-secret")

# Define config values to mount the datalake
configs = {"fs.azure.account.auth.type": "OAuth",
           "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
           "fs.azure.account.oauth2.client.id": f"{client_id}",
           "fs.azure.account.oauth2.client.secret": f"{client_secret}",
           "fs.azure.account.oauth2.client.endpoint": f"https://login.microsoftonline.com/{tenant_id}/oauth2/token"}

# define the mount function
def mount_adls(container_name):
  dbutils.fs.mount(
    source = f"abfss://{container_name}@{storage_account_name}.dfs.core.windows.net/",
    mount_point = f"/mnt/{storage_account_name}/{container_name}",
    extra_configs = configs)

# Mount the datalake
mount_adls("raw")
```

Y para probar, podemos ver contenido existente en el container que ya tengo en mi datalake, llamado "raw", donde he subido un fichero (`%fs ls /mnt/databricksecb/raw`)

Finalmente, todo OK. 

![mount_test](/img/posts_published_in_other_sites/montar_datalake_databricks/mount_test.png)


A partir de ahora, este cluster ya tiene el volumen montado para poder trabajar con normalidad. 