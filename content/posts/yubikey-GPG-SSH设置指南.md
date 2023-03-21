---
title: yubikey GPG SSH设置指南
date: 2022-11-28 17:12:30
tags:
---

# 明确你的目的

yubikey可以用于多种用途，首先需要明确的是自己需要哪些用途，而这些用途又有着多种实现方法，并且实现的难度被使用的系统影响

最主要的用途大概是GPG认证加解密、SSH认证、OATH认证、无密码登录系统、密码管理器认证等

# 选择实现目的的方式

SSH认证可以通过GPG、FIDO2的SSH密钥(ed25519-sk和ecdsa-sk)、基于PIV([WinCrypt SSH](https://github.com/buptczq/WinCryptSSHAgent/))等方式实现，但是在不同系统上实现难度不同。个人推荐使用FIDO2，是支持程度最高的

|       | Windows | macOS | Linux |
| ----- |:-------:|:-----:|:-----:|
| GPG   | 难       | 中     | 易     |
| FIDO2 | 易       | 易     | 易     |
| PIV   | 易       | 中     | 中     |

注：目前使用Cloudflare Tunnel代理的SSH无法和GPG方式共存

# GPG设置

#### Windows

[Gpg4win](https://www.gpg4win.org/)用于设置PowerShell和Cmd下的GPG，[Git Bash](https://git-scm.com/)默认自带GPG套件，与PowerShell和Cmd完全独立。下载安装即可，但是注意密钥等配置两者不共通

#### macOS

`brew install gnupg pinentry-mac`(干净无GUI)

`brew install gpg-suite-no-mail --cask`(带有GUI，安装完整套件，不带付费的邮件插件)

`brew install gpg-suite --cask`(带付费的邮件插件)

三者任选其一

#### Linux

默认自带

# SSH各种实现方式

## FIDO2

#### Windows

若要在PowerShell和Cmd下使用SSH，Windows默认的OpenSSH版本过低，需要手动安装。下载[OpenSSH](https://github.com/PowerShell/Win32-OpenSSH/)的msi安装包进行安装，配置环境变量Path，增加一条`C:\Program Files\OpenSSH`(这是新安装的SSH的路径)，删除原本的SSH的路径

Git Bash默认自带

IDE方面，IDEA等JetBrains家的软件使用的SSH库与系统的无关，新版本支持此方式

SSH客户端方面，目前只有[PuTTY-CAC](https://risacher.org/putty-cac/)和[termius](https://termius.com/)付费版支持此方式

注意必须使用管理员模式打开软件才能使用

#### macOS

macOS自带的OpenSSH在编译时去除了FIDO2的支持，通过`brew install openssh`即可覆盖。如果不行可以尝试修改环境变量`export PATH=$(brew --prefix openssh)/bin:$PATH`

#### Linux

默认自带

## PIV

Windows下安装[WinCrypt SSH](https://github.com/buptczq/WinCryptSSHAgent/)即可，macOS和Linux默认自带

个人认为设置的步骤比FIDO2更加烦琐，且对软件的支持未知，不推荐使用

## GPG

#### Windows

IDE方面，JetBrains家不支持此方式

SSH客户端方面，只有PuTTY-CAC可能以后会支持，其余都不支持

在Git Bash上配置和在Linux上配置相同

PowerShell和Cmd下需要使用[wsl-ssh-pageant](https://github.com/benpye/wsl-ssh-pageant/)

个人建议别用

#### macOS

[参考](https://jms1.net/yubikey/make-ssh-use-gpg-agent.md)，执行以下命令后重启电脑即可

```bash
cd ~/Library/LaunchAgents
curl -O https://jms1.net/yubikey/net.jms1.gpg-agent.plist
curl -O https://jms1.net/yubikey/net.jms1.gpg-agent-symlink.plist
```

无法和JetBrains家的IDE使用

#### Linux

配置`~/.gnupg/gpg-agent.conf`，增加`enable-ssh-support`
配置`~/.bashrc`，增加

```bash
export GPG_TTY="$(tty)"
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
gpgconf --launch gpg-agent
```

# 虚拟机下使用

以VMware为例，在虚拟机的vmx文件中添加

```
usb.generic.allowHID = "TRUE"
usb.generic.allowLastHID = "TRUE"
```

# 密码管理器认证

个人采用KeePassXC，在YubiKey Manager的OTP选项中，设置slot1为Challenge-response，用于验证。设置slot2为Static password，输入KeePassXC的密码。这样就可以通过长按yubikey实现打开密码数据库的操作。但是为了安全性考虑，推荐不添加slot2的静态密码
