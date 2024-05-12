---
layout: post
title:  "Copying files from docker container to host"
date:   2021-03-24 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: WSL2 DataNinja
---

Suppose that you are working from a stateless docker container in your dev environment (no mounted volumes) and you ended with some changes in files that you don´t want to loose. 

One easy way to do that is to use the `docker cp` utility to get the data out of the container.

<!--end_excerpt-->

## Search for the container

The first thing to do is to get the id of your container

```bash
 enrique  laptop  ~  $  docker ps
CONTAINER ID   IMAGE                            COMMAND                   CREATED       STATUS       PORTS
       NAMES
f13dafbb392c   database_frameworktsqldatabase   "/bin/sh -c \"/tmp/da…"   2 hours ago   Up 2 hours   0.0.0.0:14330->1433/tcp   database_frameworktsqldatabase_1
735742e3ff08   tsqlgenerator:dev                "tail -f /dev/null"       2 hours ago   Up 2 hours
       TsqlGenerator
 enrique  laptop  ~  $ 
```

In my case, the container i´m working on right now is the **f13dafbb392c**

## Copy the file 

In my case, I was working from a stateless docker container and during my unit tests i detected some "data" mistakes that I wanted to fix manually (you know...quick fix :). I then ended with a new backup inside a sateless container...so my next thing is to extract that backup out of my container as soon as possible. 

To copy the file, you only need to know:
- ContainerId
- full path to file
- where you want to extract that file

```bash
docker cp $ContainerId:$fullPathToFile $outputDirectory
```

### Copy command

In our case scenario, the operation will be: 
```bash
docker cp f13dafbb392c:/var/tmp/CMCalidad-with-data2.bak .
```