---
layout: post
title:  "Cómo aplicar un commit concreto de una rama en otra"
date:   2020-09-10 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: GitHub
---

Es algo poco deseable, pero en ocasiones pasa...verdad? :) De repente estás desarrollando en una rama, aplicas un FIX crítico que debes llevarte de inmediato a otra rama en la que algun compañero está trabajando, o viceversa. Cuando hablamos de 1 o 2 cambios, algo tonto, pues a veces acabamos antes directamente aplicando nosotros el fix, pero cuando la cosa se complica porque son decenas de pequeños cambios, nuevos assets,... y no tienes el control real de lo que hablas, te interesa saber cómo hacerlo. 

<!--end_excerpt-->

## git cherry-pick

Nuestro amigo en este caso se llama git [cherry-pick](https://git-scm.com/docs/git-cherry-pick). 

El comando require el hash del commit que quieras llevarte. Yo soy bastante comodón y suelo usar el plugin de vscode llamado [mhutchie.git-graph](https://github.com/mhutchie/vscode-git-graph).

En este caso, hago checkout en la rama que tiene el commit que busco y me anoto el **commit hash**:

```git
git checkout RamaConTuCommit
```
Ahora busco a ver el commit hash que quiero llevarme a otro lado:

![git cherry-pick](/img/posts/cherry-pick/cherry-pick1.png)

Una vez lo tengo claro, lanzo el comando cherry-pick **sobre la rama A LA QUE QUIERO TRAERMELO**

```bash
git checkout RamaDondeQuieroTraerElCommit
git pull 
# git cherry-pick COMMIT-HASH
git cherry-pick a4045a23e15dff7d6daf4e0c5a77d6d7954c89d0
```

Una vez lanzado, tendrás que reparar todos los conflictos que tengas.

![cherry-pick2](/img/posts/cherry-pick/cherry-pick2.png)

En mi caso tengo varios conflictos como puedes ver. la mayoria de ellos son prácticamente porque se trata de ficheros nuevos, que por **defecto entiende que debe eliminar**.

![git-merge](/img/posts/cherry-pick/git-merge.png)

Pero yo no quiero eliminar, sino añadir, por lo que toca hacer un 

```bash
git add .  # básicamente sobre la carpeta que tiene todo lo nuevo en mi caso
```

Y finalmente hacer un commit de todo y su correspondiente push, con lo que las 2 ramas quedan con los cambios aplicados y obviamente cada una todavia sigue su propio camino.

![git-graph](/img/posts/cherry-pick/git-graph.png)