---
layout: post
title:  "Docker images of SQL server samplesdb updated"
date:   2020-09-29 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: SqlServer WSL2 Docker Kubernetes GitHub
---
If you are like me, and you don´t have SQL Server installed locally on your dev laptop thanks to docker and the [wsl2](https://devblogs.microsoft.com/commandline/wsl2-will-be-generally-available-in-windows-10-version-2004/)... you will love this. I´ve just updated the [docker images that contains all the samplesdb database](https://hub.docker.com/repository/docker/enriquecatala/mssql-server-samplesdb) with the lastest version of SQL Server.

<!--end_excerpt-->

This is the [official Docker Hub repository](https://hub.docker.com/repository/docker/enriquecatala/mssql-server-samplesdb) where you will find the images. 


## Featured Tags

| Version 	| Tag  	| Note |
|---------	|------	|------ |
| 2019 | 2019-latest | Based on 2019-CU6-ubuntu-16.04 | 
| 2019 | 2019-CU6-ubuntu-16.04 |   	
| 2019 | 2019-CU4-ubuntu-16.04 |   	
| 2019 | 2019-ctp2.5-ubuntu| DEPRECATED |
| 2017 | 2017-latest | Based on 2017-CU22-ubuntu-16.04|
| 2017 | 2017-CU22-ubuntu-16.04 | |
| 2017 | 2017-CU20-ubuntu-16.04 | |
| 2017 | 2017-latest-ubuntu | DEPRECATED |

>NOTE: Tags are based on the [official MS repository](https://hub.docker.com/_/microsoft-mssql-server)

## Video

For more info, please take a look to this video (Spanish)

<iframe width="560" height="315" src="https://www.youtube.com/embed/ULL5nntWn1A" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

And go to the official [GitHub](https://github.com/enriquecatala/) repository where you will find the source code of [mssql-server-samplesdb](https://github.com/enriquecatala/mssql-server-samplesdb) 
