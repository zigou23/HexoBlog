---
title: 关于GitHub desktop的问题
date: 2023-04-13 09:30:00
tags: issues
cover: 
---

# 关于 GitHub desktop 的问题

> 打开GitHub desktop时，鼠标会卡顿/延迟一段时间 ( 大概十几秒，具体表现为鼠标延迟 )，而且没法操作任何应用

轻薄本gpu一般不怎么能使用硬件加速，关闭它就好了。

具体操作为打开cmd，输入

```powershell
set GITHUB_DESKTOP_DISABLE_HARDWARE_ACCELERATION=1
```

然后重新启动机器就好了

