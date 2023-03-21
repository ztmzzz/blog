---
title: 破解sandboxie-plus
date: 2022-07-25 21:33:10
tags:
---

# 前提准备

1. 拥有可用于驱动签名的证书或者{% post_link Winodws安装自签名驱动 %}

2. 安装Visual Studio 2019，勾选使用C++的桌面开发

3. 安装Windows SDK，对应WDK版本

4. 安装Windows WDK, Windows 10, version 2004 https://go.microsoft.com/fwlink/?linkid=2128854

5. 安装QT5和VS的QT插件(可选，编译界面用)(QT6应该也行)

6. 在VS中安装带有Spectre缓解的C++ MFC

7. 从[GitHub](https://github.com/sandboxie-plus/Sandboxie)上下载最新的安装包和对应的源代码

8. 使用安装包安装

# 编译

1. 用VS打开`Sandboxie\SandboxDrv.sln`

2. 选择`verify.c`

3. 修改`KphVerifySignature`函数，在函数开头直接`return 0;`

4. 选择`Release,x64`，编译出`SbieDrv.sys`

# 驱动签名

在VS的`Developer Command Prompt`中执行

```
signtool sign /v /a /f 证书.pfx /p "证书密码" SbieDrv.sys
```

# 覆盖驱动

重命名原始的`SbieDrv.sys`为其他名字，替换为修改过的`SbieDrv.sys`，然后重启既可

# 输入捐赠证书

例如：

```
NAME: 123
LEVEL: CONTRIBUTOR
DATE: 01.01.2200
SIGNATURE: 1111
```
