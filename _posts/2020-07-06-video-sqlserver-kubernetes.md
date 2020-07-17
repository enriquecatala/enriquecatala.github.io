---
layout: post
title:  "Video: Desplegar SQL Server personalizado en Kubernetes OnPremises"
date:   2020-07-06 00:00:00 +0200
tipue_search_active: true
excerpt_separator: <!--end_excerpt-->
tags: YouTube SqlServer WSL2 Docker Kubernetes GitHub
---

En este vídeo voy a enseñar cómo desplegar una imagen de SQL Server 2019 personalizada contra nuestro Kubernetes baremetal corriendo en entorno virtualizado.

[Desplegar SQL Server personalizado en Kubernetes OnPremises](https://youtu.be/ZhoRuib2JLc)

<iframe width="560" height="315" src="https://www.youtube.com/embed/ZhoRuib2JLc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

.<!--end_excerpt--> 

En el vídeo vamos a partir de lo que aprendimos en el vídeo ["Crea tu propia imagen de SQL Server para docker"](https://youtu.be/9M6Ewpcfw9I), aprovechándonos de dicha imagen creada, para desplegarla contra nuestro entorno kubernetes baremetal.

El proceso de registro es bastante sencillo, partiendo del hecho que en el [anterior video](https://youtu.be/9M6Ewpcfw9I) ya registramos nuestra imagen personalizada y ahora "solo" tenemos que ponerla a funcionar contra nuestro orquestador kubernetes.
