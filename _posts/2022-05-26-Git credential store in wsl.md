---
layout: post
title:  "Git credential store in WSL"
date:   2022-05-26 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Dev OS GitHub WSL2
---

For me, Windows11 is the best shell of Linux. I´m using it in my home and work and for that reason I´m using WSL2 to run my Linux and dev station (python, scala,...), so there is a trick to avoid the problem of the Git credential store in WSL2. The trick is to configure the credential helper of the git inside WSL2 as the windows11 credential helper.


<!--end_excerpt-->

To do this, just run the following command in the terminal or your WSL2 shell:

```bash
git config --global credential.helper "/mnt/c/Program\\ Files/Git/mingw64/libexec/git-core/git-credential-manager-core.exe"
```

Now, you will be storing the same credentials of windows and linux

