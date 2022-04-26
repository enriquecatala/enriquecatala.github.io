---
layout: post
title:  "Windows Terminal bonito"
date:   2021-05-13 00:00:00 +0200
comments: true
tipue_search_active: false
excerpt_separator: <!--end_excerpt-->
tags: Dev OS
---

Si usas [Windows Terminal](https://github.com/microsoft/terminal) y tiras mucha linea de comando, en este post vamos a ver cómo hacer que nuestro terminal se vea estupendamente :)

![cascadia code PL](/img/posts/pretty-terminal/cascadia-code2.png)
![nerd fonts](/img/posts/pretty-terminal/nerdfonts.png)

<!--end_excerpt-->

>NOTA: Post basado en [How to make a pretty prompt in Windows Terminal](https://www.hanselman.com/blog/how-to-make-a-pretty-prompt-in-windows-terminal-with-powerline-nerd-fonts-cascadia-code-wsl-and-ohmyposh)


## 1) Instala Windows Terminal

Descargalo de [aqui](https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701)

## 2) Configuracion PowerShell

Para powershell

### Abre una ventana powershell

![x](/img/posts/pretty-terminal/1.png)

### Instala oh-my-posh

```powershell
winget install JanDeDobbeleer.OhMyPosh
```

Los temas cuelgan por defecto de esta variable _$env:POSH_THEMES_PATH_

### Activa ejecion de powershell firmados remotamente a nivel usuario

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Edita el fichero de powershell de arranque

Edita el fichero de arranque de powershell que se encuentra en la variable $PROFILE

```powershell
notepad $PROFILE
```
añade esta línea `oh-my-posh init pwsh | Invoke-Expression`

## 3) Configuracion WSL2

Toca ahora el turno de tu distro favorita. En mi caso uso Ubuntu y la forma de activarlo es:

### Instalar oh-my-posh

```bash
wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O /usr/local/bin/oh-my-posh
chmod +x /usr/local/bin/oh-my-posh
```

### Descargar los [temas](https://ohmyposh.dev/docs/themes/)

```bash
mkdir ~/.poshthemes
wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/themes.zip -O ~/.poshthemes/themes.zip
unzip ~/.poshthemes/themes.zip -d ~/.poshthemes
chmod u+rw ~/.poshthemes/*.json
rm ~/.poshthemes/themes.zip
```

### Activar el tema

Edita tu fichero _~/.bashrc_ y añade la siguiente línea:

```bash
eval "$(oh-my-posh --init --shell bash --config ~/.poshthemes/jandedobbeleer.omp.json)"
```

>NOTA: Elige tu [tema](https://ohmyposh.dev/docs/themes/). En la imagen ves el de _jandedobbeleer_

## 3) Instala una fuente de texto adecuada

- Descarga [Cascadia-Code](https://github.com/microsoft/cascadia-code/releases) 
- Descarga las [Nerd fonts](https://ohmyposh.dev/docs/configuration/fonts)
- Instala _CascadiaCodePL.ttf_ 

![CascadiaCodePL.ttf](/img/posts/pretty-terminal/cascadia-code.png)

- Instala _Meslo LG S Regular Nerd Font Complete Windows Compatible.ttf_
- Añade en el fichero Settings.json de Windows Terminal la siguiente propiedad:

```json 
"fontFace":  "MesloLGS Nerd Font"
```

De esta forma, por ejemplo la sección de powershell quedaría algo así:

```json
 {
   // Make changes here to the powershell.exe profile.
   "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
   "name": "Windows PowerShell",
   "commandline": "powershell.exe",
   "fontFace":  "MesloLGS Nerd Font",
   "hidden": false
}
```

Tambien puedes hacerlo con el nuevo GUI

![settings nerd fonts windows terminal](/img/posts/pretty-terminal/settings-font.png)


- Añade en tu visual studio code la siguiente configuración tambien `"terminal.integrated.fontFamily": "MesloLGS Nerd Font"`

>NOTA: [Aqui](https://www.nerdfonts.com/) tienes un montón de fuentes que puedes añadir. Solo recuerda utilizar una que tenga soporte para _PowerLine Glyphs_

En este punto, ya deberias ver algo así:

![cascadia code PL](/img/posts/pretty-terminal/cascadia-code2.png)
![nerd fonts](/img/posts/pretty-terminal/nerdfonts.png)

![code-pretty-terminal](/img/posts/pretty-terminal/visual_studio_pretty_terminal.png)

## Mi configuración

La configuración que estoy utilizando yo es:
- Fuente _"MesloLGS NF"_
- Tema personalizado que puedes encontrar como json [aqui](/assets/files/oh-my-posh-v3-v2.json)
  - basado en el tema _"slim"_

> eval "$(oh-my-posh --init --shell bash --config ~/.poshthemes/oh-my-posh-v3-v2.json)"
