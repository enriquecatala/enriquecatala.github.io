---
layout: post
title:  "Conectar vscode con databricks"
date:   2021-07-21 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Databricks Python Dev AI Azure DataPlatform
---

En este post rápido voy a repasar cómo de facil es **conectar** nuestro entorno local con **vscode** hacia nuestros clusters de **databricks**.

![databricks1](/img/posts/configurar-vscode-con-databricks/databricks1.png)

<!--end_excerpt-->

## Descargar plugin 

Descárgate e instala este [plugin](https://marketplace.visualstudio.com/items?itemName=paiqo.databricks-vscode)

## Genérate un PersonalAccessToken

Para generar el PAT, puedes irte dentro de tu Databricks a la sección de 


![Databrick´s personal access token](/img/posts/configurar-vscode-con-databricks/personal-access-token1.png)

Y luego pinchar sobre "_Generate New Token_"

![Databrick´s personal access token](/img/posts/configurar-vscode-con-databricks/personal-access-token.png)

Una vez obtengas el token, ya estás listo para añadir la conexión a tu cluster

## Añade la conexión a tu cluster

Para añadir la conexión a tu cluster, necesitas los siguientes datos:

- Root URL de tu cluster
  - La puedes encontrar en la propia URL de tu cluster en el navegador o en el propio recurso Databricks de Azure, si como es tu caso tambien has utilizado Azure Databricks como yo

    ![root url](/img/posts/configurar-vscode-con-databricks/root-url.png)

- Nombre para mostrar en tu lista de conexiones
  - En mi caso he utilizado como nombre SentinelML
- Personal Access Token ([generado ya](#genérate-un-personalaccesstoken))
  - El token que obtuviste en el [paso anterior](#genérate-un-personalaccesstoken)
- Ruta local donde vas a querer sincronización de Notebooks entre tu cluster y tu entorno local
  - En mi caso se trata de una ruta local y en la que voy a tener git conectado

Finalmente solo tienes que entrar en la configuración de VScode y rellenar pertinentemente los datos que te solicita como _"Mandatory"_ 
  
![databricks settings](/img/posts/configurar-vscode-con-databricks/databricks-settings.png)


Y ya lo deberias ver todo correctamente:

![databricks1](/img/posts/configurar-vscode-con-databricks/databricks1.png)
