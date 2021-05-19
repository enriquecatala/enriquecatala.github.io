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

### Instala los siguientes módulos

```powershell
Install-Module posh-git -Scope CurrentUser
Install-Module oh-my-posh -Scope CurrentUser
```

Si además utilizas PowerShell Core, activa esto tambien

```powershell
Install-Module -Name PSReadLine -AllowPrerelease -Scope CurrentUser -Force -SkipPublisherCheck
```

### Activa los módulos por defecto

Para activar los módulos anteriores por defecto al arrancar powershell, ejecuta `notepad $PROFILE` 

![powershell profile](/img/posts/pretty-terminal/notepad.png)

y añadele estas líneas:

```powershell
Import-Module posh-git
Import-Module oh-my-posh
Set-PoshPrompt -Theme Paradox
```

y este es el aspecto que vas a tener ahora:

![pretty-terminal1](/img/posts/pretty-terminal/without-fonts.png)

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

- Descarga las [Nerd fonts](https://ohmyposh.dev/docs/fonts)

- Instala _CascadiaCodePL.ttf_ 

![CascadiaCodePL.ttf](/img/posts/pretty-terminal/cascadia-code.png)

- Instala _Meslo LG S Regular Nerd Font Complete Windows Compatible.ttf_

- Añade en el fichero Settings.json de Windows Terminal la siguiente propiedad:

```json 
"fontFace":  "Cascadia Code PL"
```

De esta forma, por ejemplo la sección de powershell quedaría algo así:

```json
 {
   // Make changes here to the powershell.exe profile.
   "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
   "name": "Windows PowerShell",
   "commandline": "powershell.exe",
   "fontFace": "Cascadia Code PL",
   "hidden": false
}
```

Tambien puedes hacerlo con el nuevo GUI

![settings nerd fonts windows terminal](/img/posts/pretty-terminal/settings-font.png)


- Añade en tu visual studio code la siguiente configuración tambien `"terminal.integrated.fontFamily": "Cascadia Code PL"`

>NOTA: [Aqui](https://www.nerdfonts.com/) tienes un montón de fuentes que puedes añadir. Solo recuerda utilizar una que tenga soporte para _PowerLine Glyphs_

En este punto, ya deberias ver algo así:

![cascadia code PL](/img/posts/pretty-terminal/cascadia-code2.png)
![nerd fonts](/img/posts/pretty-terminal/nerdfonts.png)

![code-pretty-terminal](/img/posts/pretty-terminal/visual_studio_pretty_terminal.png)

## Mi configuración

La configuración que estoy utilizando yo es:
- Fuente _"MesloLGS NF"_
- Tema _"slim"_