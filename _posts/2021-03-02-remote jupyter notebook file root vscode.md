---
layout: post
title:  "Remote jupyter notebook file root in vscode"
date:   2021-03-02 00:00:00 +0200
tipue_search_active: true
excerpt_separator: <!--end_excerpt-->
tags: Dev Data AI
---

When we use jupyter extensions in VScode and try to run a line of code that requires context of relative path, the default behaviour in vscode is different that the one we have with jupyter-notebooks. You can easily see this when comparing the output of a pwd command, opening the same notebook connected to the same jupyter notebook server... 

## VScode 

VSCode connected to the same jupyter-notebook server (remote)

![vscode-notebook-file-root](/img/posts/notebook-file-root/vscode-notebook-file-root.png)

<!--end_excerpt-->

## Jupyter-notebook web

Jupyter notebook server (same notebook opened from vscode in the same remote server)

![jupyter-notebooks-file-root](/img/posts/notebook-file-root/jupyter-notebook-file-root.png)

As you can see, the vscode execution fails because it´s trying to open a module that doesn´t found, since it´s searching in the wrong directory.

## Change VSCode notebook file root behavior 

<mark>NOT WORKING FOR REMOTE JUPYTER KERNELS</mark>

To make VSCode notebook run as jupyter-notebook server, you need to open settings in VSCode and change "notebook file root", from _\${workspaceFolder}_ to _\${fileDirname}_ 

So the config file must be like this:

![notebook-file-rootfiledirname](/img/posts/notebook-file-root/notebook-file-rootfiledirname.png)




## Fix for remote jupyter kernel: Force the path with python :)

But the things get complicated when you are dealing with remote jupyter notebooks, because it seems that we _still_ don´t have any solution. In my case, when I work with tensorflow, I´m always using my prepared tensorflow docker image containing all the libraries and configurations...so this requires me to connect the vscode remotely to the container...and the past solution mentioned earliear doesn´t work :(

The only solution I was able to use is to manually change the working directory: 

```python
import os
try:
    os.chdir(os.path.join(os.getcwd(), '/tf/notebooks/DNN/')) # '.' if the path is to current folder
    #print(os.getcwd())
except:
    pass
```

>NOTE: For more info, please check this https://github.com/microsoft/vscode-remote-release/issues/43