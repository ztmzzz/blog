---
title: Voxeldance Tango无限试用
date: 2022-10-03 23:52:46
tags:
draft: true
---

Voxeldance Tango是一款非常好用的3D打印切片软件，但是它采取订阅制且价格过于昂贵。不过好在它提供了每个用户的15天免费试用，这就带来了白嫖的方法。

经过查看，发现Tango是通过读取注册表`HKLM\SOFTWARE\Microsoft\Cryptography`的`MachineGuid`生成机器码的，因此只需要修改这项就能做到无限试用。以下是一个简单的bat脚本，可以自动完成修改注册表的工作。

```bat
%1 start "" mshta vbscript:CreateObject("Shell.Application").ShellExecute("cmd.exe","/c ""%~s0"" ::","","runas",1)(window.close)&&exit
set /a a=%random%%%(99999999-10000000+1)+10000000
set /a b=%random%%%(9999-1000+1)+1000
set /a c=%random%%%(9999-1000+1)+1000
set /a d=%random%%%(9999-1000+1)+1000
set /a e=%random%%%(999999-100000+1)+100000
set /a f=%random%%%(999999-100000+1)+100000
reg add "HKLM\SOFTWARE\Microsoft\Cryptography" /v "MachineGuid" /t REG_SZ /d "%a%-%b%-%c%-%d%-%e%%f%" /f
del /f "C:\ProgramData\Voxeldance Tango\VDTangoCache.db"
```
