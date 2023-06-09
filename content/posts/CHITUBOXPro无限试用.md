---
title: CHITUBOXPro无限试用
date: 2022-08-17 15:27:49
tags:
draft: true
---

代码在[GitHub - ztmzzz/CHITUBOXPro-trial: trial CHITUBOXPro every 7 days](https://github.com/ztmzzz/CHITUBOXPro-trial)

# 介绍

CHITUBOXPro是一个很好用的切片工具，并且提供了7天的免费试用。并且账号的创建不需要进行邮箱的验证(自建poste.io可解)，根据hwid判断机器(hwid spoofer可解)，因此可以无限创建账号来无限试用。

使用selenium来注册账号，因为只是个人使用所以无所谓效率。修改配置文件让CHITUBOXPro认为这是新的机器，pyautowin自动操作软件进行试用流程

# "破解"记录

## 第一天

买了新打印机正在测试的过程中，偶然间想到CHITUBOXPro有更多的高级功能，而且貌似还有试用时间，那么为什么不尝试搞个无限试用呢？

说干就干，先去官网尝试了一下，发现只要注册个账号就有7天的试用时间，想着这么简单随便花个几小时弄一下全自动试用就行。然后想着还有验证邮件，不如自建一个邮件服务器，选择了poste.io，并且查好了api，测试了一下没啥问题，可以先放一边。

到群里问了一下有没有这个需求的，结果群友告诉说是和机器码绑定的，注册了一个新账号果然如此，一台机器只能试用一次。

机器码很简单，掏出VMware看看能不能改点东西，一眼就看到了`bios.uuid`，这不就是bios的序列号么，改了这玩意肯定识别成新的机器了。试了一下居然机器码没变，用`geek`强制删除发现在`AppData\Local\CHITUBOXPro`下有个`machineInfo.cfg`，这不就好办了，删掉重新打开软件发现机器码改变，可以继续试用。

刚准备到群里发一下好消息，给大伙送个福利，发现有人说有空做个loader搞个全自动的。我仔细想了一下，这个方案要求使用虚拟机，而且每次切换机器码都要重启虚拟机，确实是个麻烦的方案，需要深刻检讨一下自己思考不全面的问题。那么现在就要求不重启系统改变机器码，一查发现有很多种方法。不过和做外挂的需求差不多，找到了`hwid spoofer`，用驱动直接修改，哼哧哼哧一顿找，成功骗过了`wmic bios get serialnumber`，能够返回修改的数据。

## 第二天

也不知太激动了还是睡得晚导致出现幻觉，以为昨天已经做好测试能骗过CHITUBOXPro，以为自己解决了核心问题开开心心的去搞自动化试用了。不知道是QT的问题还是CHITUBOXPro的问题，账号密码的输入框找得到，但是登录按钮完全不知道在哪，inspect也显示都是静态。无奈之下只能选择模拟鼠标进行点击。因为VMware下的3D加速会导致界面透明，所以还不能用图片识别的方式，无奈只能手动输入偏移量。

回到驱动的部分，先是拷贝了启动驱动的代码进行测试，发现总是会返回驱动启动失败，再加上自己对于Windows编程一窍不通，干脆直接写cmd脚本，再用Python调用。最终结果是Python调用vbs脚本，vbs用管理员权限运行cmd脚本。

差不多都完成了，先总体运行一次尝试一下，看起来很棒，驱动执行也很完美，就是在运行第二次的时候为什么软件没法试用了呢？

打开`machineInfo.cfg`一看，压根就没变，但是我残存的记忆告诉我昨天是做好了测试能用的，于是又是几次测试，发现完全没变化。在排除了多个因素之后，我跑到回收站一看，笑死，昨天压根没测试过。

看来不是根据`wmic`获取的机器码，拿出ProcessMonitor一看，发现其实访问了很多的注册表，并且也同时调用了wmic，查了一下发现一些注册表是和hwid有关的。看起来应该是双管其下。于是我保存了写入`machineInfo.cfg`前所有的记录，修改VMware的`bios.uuid`，对这2份记录进行对比，发现数据基本没什么区别，区别的部分和机器码也没啥关系，这下搞不明白了，想着可能这次尝试失败了

## 第三天

早上起来破罐子破摔，想着能不能替换`machineInfo.cfg`来改变机器码，居然可行。这真的是离谱，每次启动都要读注册表和wmic，然后也不重新校验一下，就让人随便改。在对比了几份生成的`machineInfo.cfg`后，测试得到前面一部分固定，后面一部分随便改，机器码会对应改变。

那么这次"破解"就算是差不多完成了，优化优化程序，就做到了在不重启的情况下无限试用，并且不仅限于虚拟机。总计花费3天时间。
