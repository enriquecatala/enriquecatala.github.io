---
layout: post
title:  "Libera espacio del fichero ext4.vhdx"
date:   2021-04-15 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Docker Dev
---

Si utilizas #wsl2 para desarrollar y acabas generando contenedores grandes, verás crecer fuertemente tu fichero dockerfile donde se almacenan todos esos contenedores. Recientemente he estado trabajando con el modelo [GPT-NEO](https://github.com/EleutherAI/gpt-neo/) que tiene la friolera de 10.2Gb de tamaño en pesos de red neuronal...a la que he metido mis propios "artefactos" (dejemoslo ahi :). El caso es que cada etapa de generacion lleba a superar los 15Gb, haciendo que al final de la contienda mover una imagen como esta llegó a menearme 150Gb de disco.

Llegado el momento de limpiar el asunto me encontré con esto:

```bash
Total reclaimed space: 157.7GB
```

Y con esto:
![ext4.vhdx](/img/posts/clean-ext4vhd/ext4.vhdx.png)

<!--end_excerpt-->

Para liberar el espacio tendras que seguir estos pasos:
1. Parar docker
2. Ejecutar `Optimize-VHD -Path c:\path\to\data.vhdx -Mode Full`
3. Esperar :)

En mi caso lancé:

```powershell
Optimize-VHD -Path C:\Users\ecatala\AppData\Local\Docker\wsl\data\ext4.vhdx -Mode Full
```

![ext4.vhdx-2](/img/posts/clean-ext4vhd/ext4.vhdx-2.png)


