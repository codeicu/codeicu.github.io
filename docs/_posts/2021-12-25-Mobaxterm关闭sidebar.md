---
layout: post
title:  "Mobaxterm关闭sidebar"
date:   2021-12-25 12:25:38 +0800
categories: 工具
---

# MobaXterm 是个好用的工具

功能全面, 轻量, 界面还行. 但是每次打开session自动展开sidebar很烦.

# Disable MobaXterm file browser sidebar from opening automatically

While instructing users through the GUI settings is an option it is often desirable to implement the application with an already usable default configuration. These settings are stored in the MobaXterm.ini file.

## How to locate my MobaXterm configuration file?
- Well, it depends… Your MobaXterm.ini configuration file should be located:

- in the same folder as MobaXterm executable if you are using MobaXterm Portable Edition
in %MyDocuments%\MobaXterm folder if you are using MobaXterm Installer Edition
- in %MyDocuments%\MobaXterm folder if you are using MobaXterm Portable Edition and if you only have read access to the folder where the executable is

# Hodify configuration file
[WindowPos_xxxxxxxxxxxxx]
...
CompactMode=0
SidebarVisible=0
...

[SSH]
...
AutoStartSSHGUI=0
EnableSFTP=0
...

# 注意, 要在关闭MobaXterm后再修改配置文件