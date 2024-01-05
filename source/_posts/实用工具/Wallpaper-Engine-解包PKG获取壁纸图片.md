---
title: Wallpaper-Engine-解包PKG获取壁纸图片
cover: false
date: 2024-01-04 09:54:06
tags:
categories:
 - [实用工具,windows工具]
top:
coverImg:
---
解包从 wallpaper engine 下载到的桌面壁纸文件提取图片

github.com/notscuffed/repkg

下载发行版本  https://github.com/notscuffed/repkg/releases/tag/v0.2.2-alpha

{% asset_img image.png 配置图1 %}

下载完成后解压缩

在 wallpaper engine 中找到你要解包的壁纸文件-右键-点击在资源管理器中打开作

将此文件夹复制后放到根目录新建文件夹-REPKG 

{% asset_img image-1.png 配置图2 %}

然后在文件夹空白处按 shift+右键打开 powershell

{% asset_img image-2.png 配置图3 %}

三. 输入：

`.\repkg extract -o ./output D:\REPKG`

{% asset_img image-3.png 配置图4 %}

一般 JPG、PNG 图片都在文件 ..\output\*\materials\ 目录下

