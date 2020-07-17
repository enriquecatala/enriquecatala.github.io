---
layout: post
title:  "Fix for ms-screensketch protocol"
date:   2020-07-13 00:00:00 +0200
tipue_search_active: true
excerpt_separator: <!--end_excerpt-->
tags: bugfix
---

With the new Windows 10 2004 update I´ve been suffering the bug from the new ScreenSketch tool that comes by default with this new version of Windows 10.

I´ve been always using the combination of _ctrl+shift+s_ to take screenshots of my PC to easily select small parts of my desktop. This is very usefull to do quick edits specially when working with markdown...But after the windows upgrade, anytime I was trying the combination, I was receiving this error:


![ms-screenclip protocol](/img/posts/screensketch/ms-screenclip-link%20broken.png)

The problem was apparently very easy to fix...but i tried everything without success...

<!--end_excerpt-->

My fix was basically to force the OneNote 2016 screen clipping again, by uninstalling the "new" screensketch :).

## Hard fix

### 1) Uninstall screen sketch
Since ScreenSketch tool is installed by default with Windows 10, you are forced to uninstall it manually through PowerShell.

```powershell
Get-AppxPackage *Microsoft.ScreenSketch* -AllUsers | Remove-AppxPackage
```

### 2) Force OneNote 2016´s screen clipping tool

Now that you basically uninstalled the buggy tool, you can edit the shortcut key combination for OneNote 2016 (which is free) and start using it as always:

Open RegEdit.exe and edit this key:
```
HKEY_CURRENT_USER\SOFTWARE\Microsoft\Office\16.0\OneNote\Options\Other\ScreenClippingShortcutKey
```

And use any combination you want: http://www.kbdedit.com/manual/low_level_vk_list.html

I´m using **58(HEX)**, so now when I do _Win + Shift + x_ i´m basically doing what I needed to :) 




>NOTE: Please, take a look on this [post](https://answers.microsoft.com/en-us/windows/forum/all/snip-sketch-broken-for-me/136918f2-6956-4c90-9434-374a42280429) because may be you are more lucky than me :) 

## Possible workaround

A possible workaround to avoid uninstalling ScreenSketch was to force a reinstall of the tool...which at some point will recreate the protocol handler.

```powershell
Get-AppXPackage -AllUsers | Where-object {$_.name -eq "Microsoft.ScreenSketch"} | Foreach {Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"}
```

>NOTE: By the time this post has been written, this option didn´t work