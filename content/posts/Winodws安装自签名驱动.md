---
title: Winodws安装自签名驱动
date: 2022-07-25 21:33:24
tags:
---

# 参考链接

1. [Windows10-CustomKernelSigners/README.zh-CN.md at master · HyperSine/Windows10-CustomKernelSigners · GitHub](https://github.com/HyperSine/Windows10-CustomKernelSigners/blob/master/README.zh-CN.md)

2. [GitHub - valinet/ssde: SSDE is a collection of utilities that help in having Windows load your custom signed kernel drivers when Secure Boot is on and you own the system&#39;s platform key, instead of using test mode.](https://github.com/valinet/ssde)

3. [Stuck on step 2.5 (Guide to get out of the Boot Loop) · Issue #7 · HyperSine/Windows10-CustomKernelSigners · GitHub](https://github.com/HyperSine/Windows10-CustomKernelSigners/issues/7)

# 简略步骤

1. 根据链接1，操作到步骤2.4完成

2. 根据链接2，从步骤2.5开始操作直到完成

# 注意事项

1. 电脑的主板必须支持修改安全启动的Platform Key(PK)，我的华硕主板比较麻烦，需要先关闭安全启动，清除所有安全密钥，然后重启到bios，可以看到PK未加载。然后打开安全启动，加载自己的PK，不要忘了把下面的KEK等加载为主板的默认设置

2. 如果使用`ssde_enable.exe`导致电脑无限重启(一般5次以上才算)，参考链接2的办法，找一个U盘，用Rufus装个恢复镜像，从U盘进入，选择命令提示符，输入`regedit`，选择`Windows\System32\config\SYSTEM`，随便打个名字，设置`Setup/CmdLine`为空，`Setup/SetupType`为0

3. 加载注册表的时候可能出现`加载配置单元`灰色的情况，此时点击`HKEY_LOCAL_MACHINE`既可。

4. 如果选择用恢复镜像手动设置`HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\CI\Protected\Licensed`为1的话，一般来说要尝试3次以上才能成功

5. 注意操作的顺序，我的电脑必须先使用`ssde_query.exe`查看返回值为1后，再启动ssde服务，不能把顺序颠倒了。但是虚拟机可以把顺序颠倒。最好先把`ssde.sys`签名后再操作

6. 记得把`ssde.sys`放到`%windir%\system32\drivers\`中

7. 需要在命令行中运行`ssde_query.exe`才能看到输出，系统会很快的修改注册表的数值，因此要一开机立马查询，只要输出是1就是成功了，可以执行
   
   ```
   sc create ssde binpath=%windir%\system32\drivers\ssde.sys type=kernel start=boot error=normal
   ```

8. 可以使用`sc delete ssde`,`sc query ssde`来删除和查询ssde服务(仅限CMD)

# 详细步骤

1. 创建根证书localhost-root-ca
   
   ```powershell
   $cert_params = @{
    Type = 'Custom'
    Subject = 'CN=Localhost Root Certification Authority'
    FriendlyName = 'Localhost Root Certification Authority'
    TextExtension = '2.5.29.19={text}CA=1'
    HashAlgorithm = 'sha512'
    KeyLength = 4096
    KeyAlgorithm = 'RSA'
    KeyUsage = 'CertSign','CRLSign'
    KeyExportPolicy = 'Exportable'
    NotAfter = (Get-Date).AddYears(100)
    CertStoreLocation = 'Cert:\LocalMachine\My'
   }
   
   $root_cert = New-SelfSignedCertificate @cert_params
   ```

2. 创建内核代码证书localhost-km
   
   ```powershell
   $cert_params = @{
       Type = 'CodeSigningCert'
       Subject = 'CN=Localhost Kernel Mode Driver Certificate'
       FriendlyName = 'Localhost Kernel Mode Driver Certificate'
       TextExtension = '2.5.29.19={text}CA=0'
       Signer = $root_cert
       HashAlgorithm = 'sha256'
       KeyLength = 2048
       KeyAlgorithm = 'RSA'
       KeyUsage = 'DigitalSignature'
       KeyExportPolicy = 'Exportable'
       NotAfter = (Get-Date).AddYears(10)
       CertStoreLocation = 'Cert:\LocalMachine\My'
   }
   
   $km_cert = New-SelfSignedCertificate @cert_params
   ```

3. 创建UEFI Plaform Key证书localhost-pk
   
   ```powershell
   $cert_params = @{
       Type = 'Custom'
       Subject = 'CN=Localhost UEFI Platform Key Certificate'
       FriendlyName = 'Localhost UEFI Platform Key Certificate'
       TextExtension = '2.5.29.19={text}CA=0'
       Signer = $root_cert
       HashAlgorithm = 'sha256'
       KeyLength = 2048
       KeyAlgorithm = 'RSA'
       KeyUsage = 'DigitalSignature'
       KeyExportPolicy = 'Exportable'
       NotAfter = (Get-Date).AddYears(10)
       CertStoreLocation = 'Cert:\LocalMachine\My'
   }
   
   $pk_cert = New-SelfSignedCertificate @cert_params
   ```

4. 在`certlm.msc`的`个人\证书`中导出生成的三份证书

5. 设置UEFI KEY
   在VMware的vmx文件中加入，删除nvram文件
   
   ```
   uefi.allowAuthBypass = "TRUE"
   uefi.secureBoot.PKDefault.file0 = "localhost-pk.der" 
   ```

6. 创建内核代码证书规则
   在Windows企业版中运行以下命令得到`SiPolicy.bin`
   
   ```powershell
   New-CIPolicy -FilePath SiPolicy.xml -Level RootCertificate -ScanPath C:\windows\System32\
   Add-SignerRule -FilePath .\SiPolicy.xml -CertificatePath .\localhost-km.der -Kernel
   ConvertFrom-CIPolicy -XmlFilePath .\SiPolicy.xml -BinaryFilePath .\SiPolicy.bin
   ```

7. 签名并导入规则
   
   ```powershell
   signtool sign /fd sha256 /p7co 1.3.6.1.4.1.311.79.1 /p7 . /f .\localhost-pk.pfx /p <localhost-pk.pfx的密码> SiPolicy.bin
   mv .\SiPolicy.bin.p7 .\SiPolicy.p7b
   mountvol x: /s
   cp .\SiPolicy.p7b X:\EFI\Microsoft\Boot\
   ```

8. CustomKernelSigners持久化
   
   1. 签名`ssde.sys`
   
   ```
   signtool sign /fd sha256 /a /ac .\localhost-root-ca.der /f .\localhost-km.pfx /p <localhost-km.pfx的密码> ssde.sys
   ```
   
   2. 把`ssde.sys`放入`%windir%\system32\drivers\`中
   
   3. 使用`ssde_enable.exe`或者进行如下步骤
      
      1. 制作一个Windows安装U盘
      
      2. 从安装Windows界面进入，选择命令提示符，输入`regedit`
      
      3. 点击`HKEY_LOCAL_MACHINE`
      
      4. 点击`加载配置单元`，选择`Windows\System32\config\SYSTEM`
      
      5. 设置`HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\CI\Protected\Licensed`为1
      
      6. 正常启动电脑
   
   4. 进入系统后立刻用`cmd`打开`ssde_query.exe`结果是1则运行以下命令，否则回到2
   
   ```
   sc create ssde binpath=%windir%\system32\drivers\ssde.sys type=kernel start=boot error=normal
   ```

9. 检查结果
   `cmd`运行`sc query ssde`，检查是否运行既可