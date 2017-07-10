---
layout: post
title: Debian 的 "每日消息"
tags:
- linux
categories: linux
description: 
---

## Debian 的 "每日消息(MOTD)"

Debian/Ubuntu 可以通过 "MOTD" 来定制一些动态的启动提示，比如系统信息（IP地址、CPU占用、温度…）

参考这里 https://github.com/armbian/build/tree/e5579a0957fd7b394d3b418671b4f0f82b334be9/scripts/update-motd.d

只需要在 `/etc/update-motd.d/` 中写脚本就好

写个欢迎信息

```
toilet -f standard -F metal "My Server"
```
