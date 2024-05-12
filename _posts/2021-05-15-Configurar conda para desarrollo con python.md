---
layout: post
title:  "Configurar conda para desarrollo con python"
date:   2021-05-15 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: DataNinja Python Dev AI OS
---

Algún dia me animaré a hacer algun post/video sobre cómo hago dev y debug sobre python, pero mientras tanto, valga este post para recodar cómo preparar nuestro PC para desarrollo con python. Conda es un excelente entorno para data science, que incluye de serie y de forma gratuita 2 cosas super-utiles como son el editor "Spider" y la gestión de entornos muy cómoda. 

En este post vamos a ver rápidamente cómo configurar nuestro entorno python con conda para desarrollo data science sobre [Windows](#windows) y sobre [Linux](#linux).

![anaconda gui](/img/posts/configurar-conda/Anaconda.png)

Para configurarlo te propongo: 

<!--end_excerpt-->

# Windows

## Instalación 

- Instala [anaconda](https://www.anaconda.com/products/individual)
- Abre anaconda prompt
   
![anaconda prompt](/img/posts/configurar-conda/condaprompt.png)

- Lanza `where conda` y copia la ubicación de su ejecutable
- Abre "Advanced system settings" 

![advanced system settings](/img/posts/configurar-conda/advanced-system-settings.png)

- Añade a tu PATH el path de conda que te salga, que en mi caso es:

```bash
C:\Users\enriq\anaconda3\Library\bin
C:\Users\enriq\anaconda3\Scripts
C:\Users\enriq\anaconda3\condabin
C:\Users\enriq\anaconda3
```

- ya deberia funcionar lanzando `conda --version`

![conda-version](/img/posts/configurar-conda/conda-version.png)

- Asegurate que tienes la última version de anaconda `conda update anaconda`

```bash
❯ conda update anaconda
Collecting package metadata (current_repodata.json): done
Solving environment: done

# All requested packages already installed.
```

- Actualiza spider a la version 5

```bash
conda install spider=5.0.0
```

## Aislar entornos de trabajo 

Es mas que interesante crear entornos de trabajo para cada proyecto. Esto es útil por ejemplo para poder tener diferentes versiones de python/librerias que puedan ser incompatibles entre ellas.

Por ejemplo, si quisiéramos crearnos un entorno para trabajar con la IA [GPT-Neo](https://github.com/EleutherAI/gpt-neo), que en el momento de escribir estas líneas está únicamente disponible en la rama main de [huggingface](https://huggingface.co/) (no en su paquete oficial todavía)

### Crear un entorno

```bash
conda create --name gptneo python=3.8.8
```
Cuyo output...

```bash
❯ conda create --name gptneo python=3.8.8
Collecting package metadata (current_repodata.json): done
Solving environment: done

## Package Plan ##

  environment location: C:\Users\enriq\anaconda3\envs\gptneo

  added / updated specs:
    - python=3.8.8


The following NEW packages will be INSTALLED:

  ca-certificates    pkgs/main/win-64::ca-certificates-2021.4.13-haa95532_1
  certifi            pkgs/main/win-64::certifi-2020.12.5-py38haa95532_0
  openssl            pkgs/main/win-64::openssl-1.1.1k-h2bbff1b_0
  pip                pkgs/main/win-64::pip-21.0.1-py38haa95532_0
  python             pkgs/main/win-64::python-3.8.8-hdbf39b2_5
  setuptools         pkgs/main/win-64::setuptools-52.0.0-py38haa95532_0
  sqlite             pkgs/main/win-64::sqlite-3.35.4-h2bbff1b_0
  vc                 pkgs/main/win-64::vc-14.2-h21ff451_1
  vs2015_runtime     pkgs/main/win-64::vs2015_runtime-14.27.29016-h5e58377_2
  wheel              pkgs/main/noarch::wheel-0.36.2-pyhd3eb1b0_0
  wincertstore       pkgs/main/win-64::wincertstore-0.2-py38_0


Proceed ([y]/n)? y

Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate gptneo
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

### Activar el entorno

Para activar el entorno por línea de comandos puedes lanzar `conda activate gptneo`, pero si lo que quieres es desde code, hacer el cambio...la opción mas facil sería lanzar _F1->Python: select interpreter_

![select interpreter](/img/posts/configurar-conda/select-interpreter.png)

que como verás, ya automáticamente detecta el nuevo entorno:

![python gptneo](/img/posts/configurar-conda/gptneo.png)

### Instalar paquetes en el entorno

Una vez asegurados en el entorno de trabajo, ya podemos instalar nuestros paquetes con normalidad. En mi caso voy a desplegar la rama main de [git de huggingface](https://github.com/huggingface/transformers), para poder acceder a las mejoras Transformer con las que poder desplegar una AI con [GPTNeo 2.7B](https://huggingface.co/EleutherAI/gpt-neo-2.7B) (lo mas parecido ahora mismo a la versión light de [GPT-3](https://arxiv.org/abs/2005.14165))

```bash
(base) D:\git\enriquecatala.github.io>conda activate gptneo
(gptneo) D:\git\enriquecatala.github.io>pip install git+https://github.com/huggingface/transformers@master
Collecting git+https://github.com/huggingface/transformers@master
  Cloning https://github.com/huggingface/transformers (to revision master) to c:\users\enriq\appdata\local\temp\pip-req-build-2ebx9m9g
  Running command git clone -q https://github.com/huggingface/transformers 'C:\Users\enriq\AppData\Local\Temp\pip-req-build-2ebx9m9g'
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
    Preparing wheel metadata ... done
Collecting sacremoses
  Downloading sacremoses-0.0.45-py3-none-any.whl (895 kB)
     |████████████████████████████████| 895 kB 6.8 MB/s
Collecting huggingface-hub==0.0.8
  Downloading huggingface_hub-0.0.8-py3-none-any.whl (34 kB)
Collecting filelock
  Downloading filelock-3.0.12-py3-none-any.whl (7.6 kB)
Collecting tqdm>=4.27
  Downloading tqdm-4.60.0-py2.py3-none-any.whl (75 kB)
     |████████████████████████████████| 75 kB 5.1 MB/s
Collecting packaging
  Downloading packaging-20.9-py2.py3-none-any.whl (40 kB)
     |████████████████████████████████| 40 kB 2.7 MB/s
Collecting numpy>=1.17
  Downloading numpy-1.20.3-cp38-cp38-win_amd64.whl (13.7 MB)
     |████████████████████████████████| 13.7 MB ...
Collecting requests
  Downloading requests-2.25.1-py2.py3-none-any.whl (61 kB)
     |████████████████████████████████| 61 kB 3.8 MB/s
Collecting regex!=2019.12.17
  Downloading regex-2021.4.4-cp38-cp38-win_amd64.whl (270 kB)
     |████████████████████████████████| 270 kB ...
Collecting tokenizers<0.11,>=0.10.1
  Downloading tokenizers-0.10.2-cp38-cp38-win_amd64.whl (2.0 MB)
     |████████████████████████████████| 2.0 MB 6.8 MB/s
Collecting pyparsing>=2.0.2
  Downloading pyparsing-2.4.7-py2.py3-none-any.whl (67 kB)
     |████████████████████████████████| 67 kB 2.8 MB/s
Collecting idna<3,>=2.5
  Downloading idna-2.10-py2.py3-none-any.whl (58 kB)
     |████████████████████████████████| 58 kB ...
Collecting chardet<5,>=3.0.2
  Downloading chardet-4.0.0-py2.py3-none-any.whl (178 kB)
     |████████████████████████████████| 178 kB ...
Collecting urllib3<1.27,>=1.21.1
  Downloading urllib3-1.26.4-py2.py3-none-any.whl (153 kB)
     |████████████████████████████████| 153 kB ...
Requirement already satisfied: certifi>=2017.4.17 in c:\users\enriq\anaconda3\envs\gptneo\lib\site-packages (from requests->transformers==4.7.0.dev0) (2020.12.5)
Collecting six
  Downloading six-1.16.0-py2.py3-none-any.whl (11 kB)
Collecting joblib
  Downloading joblib-1.0.1-py3-none-any.whl (303 kB)
     |████████████████████████████████| 303 kB 6.4 MB/s
Collecting click
  Downloading click-8.0.0-py3-none-any.whl (96 kB)
     |████████████████████████████████| 96 kB 6.4 MB/s
Collecting colorama
  Downloading colorama-0.4.4-py2.py3-none-any.whl (16 kB)
Building wheels for collected packages: transformers
  Building wheel for transformers (PEP 517) ... done
  Created wheel for transformers: filename=transformers-4.7.0.dev0-py3-none-any.whl size=2259729 sha256=45d5f23c6d779450d66f26a1d9956684a0e88a0689b36b7e4b616dc6690149b2
  Stored in directory: C:\Users\enriq\AppData\Local\Temp\pip-ephem-wheel-cache-8am4pg42\wheels\29\97\ba\eda99aa62c8a9414a8754b0b94ed3ae80559d7c8293929140f
Successfully built transformers
Installing collected packages: urllib3, idna, colorama, chardet, tqdm, six, requests, regex, pyparsing, joblib, filelock, click, tokenizers, sacremoses, packaging, numpy, huggingface-hub, transformers
Successfully installed chardet-4.0.0 click-8.0.0 colorama-0.4.4 filelock-3.0.12 huggingface-hub-0.0.8 idna-2.10 joblib-1.0.1 
numpy-1.20.3 packaging-20.9 pyparsing-2.4.7 regex-2021.4.4 requests-2.25.1 sacremoses-0.0.45 six-1.16.0 tokenizers-0.10.2 tqdm-4.60.0 transformers-4.7.0.dev0 urllib3-1.26.4
```

Y ya está, tenemos el entorno nuevo, con huggigface transformers utilizando la última versión disponible estable, que va por delante del propio paquete oficial huggingface (que es lo que quería enseñarte yo en este caso de ejemplo)

>NOTA: Recuerda que cada entorno está totalmente aislado del resto, y esto implica que incluso el ide Spyder no estará disponible en nuestro nuevo environment, lo que quiere decir que lo tendrás que instalar ahi tambien :) 

```bash
conda install spider=5.0.0
```

# Linux

En el caso de linux, pues se puede scriptar mas facilmente para variar :)

![conda](/img/posts/configurar-conda/conda-ubuntu.png)

## Descargar anaconda

Lo primero que deberas hacer es entrar [aqui](https://www.anaconda.com/products/individual) y seleccionar la versión que quieras.

En mi caso, voy a instalarlo para Ubuntu 20.04 64 bits y python 3.8
https://repo.anaconda.com/archive/Anaconda3-2021.05-Linux-x86_64.sh

```bash
cd /tmp
# descargar
curl -L -o anaconda-installer.sh https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
# instalar
bash anaconda-installer.sh
```

En este momento te pedirá leer los términos de licencia, que obviamente deberás aceptar y te mostrará el path por defecto (que podrás cambiar donde se instalará)

```bash
Anaconda3 will now be installed into this location:
/home/enrique/anaconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/home/enrique/anaconda3] >>>
PREFIX=/home/enrique/anaconda3
Unpacking payload ...
Collecting package metadata (current_repodata.json): done
Solving environment: done

## Package Plan ##

  environment location: /home/enrique/anaconda3

  added / updated specs:
    - _ipyw_jlab_nb_ext_conf==0.1.0=py37_0
....
y aqui un largo etcétera de librerias ...
```

una vez acabada la instalación, solo queda añadir el path, que te preguntará

```bash
installation finished.
Do you wish the installer to prepend the Anaconda3 install location
to PATH in your /home/enrique/.bashrc ? [yes|no]
```

## Update conda

Es **altamente recomendable** actualizar a la última version de conda, que muy probablemente no sea la que se instale con el script anterior. Para actualizar conda en linux

```bash
conda update conda
```