---
layout: post
title:  "Ubuntu WSL2 on Windows Server 2019"
date:   2021-03-02 00:00:00 +0200
tipue_search_active: true
excerpt_separator: <!--end_excerpt-->
tags: WSL2
---

Suppose that you are working from a Windows Server 2019 and you require to make use to some linux scripts from there (and don´t want to waste your time ssh-ing to other systems). The best way to that is to install any of the supported linux distributions, like for example "Ubuntu 20.04".

## Enable  WSL on WS2019

The first thing to do is to enable WSL on your windows machine 

```bash
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```
>IMPORTANT: This requires restart the system

```bash
curl.exe -L -o Ubuntu_2004.2020.424.0_x64.appx https://aka.ms/wsl-ubuntu-2004
Add-AppxPackage .\Ubuntu_2004.2020.424.0_x64.appx
```

>NOTE: For more information, please check [this](https://docs.microsoft.com/en-us/windows/wsl/install-manual)
