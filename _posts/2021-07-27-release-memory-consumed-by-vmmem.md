---
layout: post
title:  "How to release memory consumed by vmmem WSL2"
date:   2021-07-27 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: WSL2 OS
---

There is a bug in WSL2 that causes the memory consumption of the vmmem process to not to be released. Right now, the version of WSL2 and windows i´m using is Windows 10 19043.1110 (21H1) and it´s still present.

This is what you can see when the error happens:

1) You have your WSL2 images stopped:
   
![vmmem1](/img/posts/vmmem-taking-too-much-memory/vmmem2.png)

2) There is a lot of memory ussage from the vmmem process:

![vmmem1](/img/posts/vmmem-taking-too-much-memory/vmmem1.png)


<!--end_excerpt-->

The solution for this is to simply stop the vmmem process:

```powershell
wsl --shutdown
```

Et voila :)

![vmmem3](/img/posts/vmmem-taking-too-much-memory/vmmem3.png)