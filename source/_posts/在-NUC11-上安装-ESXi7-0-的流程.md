---
title: 在 NUC11 上安装 ESXi7.0 的流程
abbrlink: 817315956
date: 2022-05-25 11:50:10
categories: 虚拟化
tags:
    - NUC
    - ESXi
    - VMware
---

虽然说现在的云服务器已经白菜价了，但对于我来说还是习惯在内网放一台测试机。毕竟本地内网的带宽更大，而且做了虚拟化后还能随时创建/销毁临时环境。

我用的是一台闲置的 NUC 11（NUC11PAHi5），静音且省电。唯一问题就是官方镜像不支持 NUC11 的网卡，所以我们需要自己打包额外网卡驱动进去，并且打包镜像需要在 Windows 环境下进行。

## 下载所需文件

首先注册一个 VMware 的官网账号，打开 [vSphere Hypervisor](https://customerconnect.vmware.com/web/vmware/evalcenter?p=free-esxi7) 产品页面。下载 `VMware-ESXi-7.0U3d-19482537-depot.zip` 压缩包（注意不是 iso 文件），同时保存产品 License Key（EXSi 是对个人用户免费的）。

然后下载 [Community Networking Driver for ESXi](https://flings.vmware.com/community-networking-driver-for-esxi) 驱动文件包，这里包含了一些非官方支持的网卡型号驱动文件。最后我们就有了这两个文件：

1. VMware-ESXi-7.0U3d-19482537-depot.zip
2. Net-Community-Driver_1.2.7.0-1vmw.700.1.0.15843807_19480755.zip

## 安装 VMware.PowerCLI

以管理员权限打开 Powershell（我是 Windows 10 系统），输入一下命令安装 VMware 的工具脚手架：

```powershell
PS C:\Software> Install-Module -Name VMware.PowerCLI
NuGet provider is required to continue
PowerShellGet requires NuGet provider version '2.8.5.201' or newer to interact with NuGet-based repositories. The NuGet
 provider must be available in 'C:\Program Files\PackageManagement\ProviderAssemblies' or
'C:\Users\guido\AppData\Local\PackageManagement\ProviderAssemblies'. You can also install the NuGet provider by running
 'Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force'. Do you want PowerShellGet to install and
import the NuGet provider now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y

Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its
InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from
'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): y
```

这里需要修改当前用户的脚本执行策略，将其设置为 `RemoteSigned`：

```powershell
PS C:\Software> Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

> PowerShell 的执行策略有以下几种：
> - 允许所有脚本: Unrestricted
> - 允许本地脚本和远程签名脚本: RemoteSigned
> - 仅允许已签名的脚本: AllSigned

## 打包自定义镜像

首先分别导入下载好的两个包：

```powershell
PS C:\Software> Add-EsxSoftwareDepot "VMware-ESXi-7.0U3d-19482537-depot.zip"

Depot Url
---------
zip:C:\Software\VMware-ESXi-7.0U3d-19482537-depot.zip?index.xml

PS C:\Software> Add-EsxSoftwareDepot ".\Net-Community-Driver_1.2.7.0-1vmw.700.1.0.15843807_19480755.zip"

Depot Url
---------
zip:C:\Software\Net-Community-Driver_1.2.7.0-1vmw.700.1.0.15843807_19480755.zip?index.xml
```

列出现有的包，选择一个作为底包并复制一个副本命名为 `ESX-NUC`，集成社区驱动并导出 iso 镜像：

```powershell
PS C:\Software> Get-EsxImageProfile

Name                           Vendor          Last Modified   Acceptance Level
----                           ------          -------------   ----------------
ESXi-7.0U3d-19482537-no-tools  VMware, Inc.    2022/3/11 15... PartnerSupported
ESXi-7.0U3d-19482537-standard  VMware, Inc.    2022/3/29 0:... PartnerSupported
ESXi-7.0U3sd-19482531-no-tools VMware, Inc.    2022/3/11 13... PartnerSupported
ESXi-7.0U3sd-19482531-standard VMware, Inc.    2022/3/29 0:... PartnerSupported

PS C:\Software> New-EsxImageProfile -CloneProfile "ESXi-7.0U3d-19482537-standard" -name "ESX-NUC"

位于命令管道位置 1 的 cmdlet New-EsxImageProfile
请为以下参数提供值:
(请键入 !? 以查看帮助。)
Vendor: whatever

Name                           Vendor          Last Modified   Acceptance Level
----                           ------          -------------   ----------------
ESX-NUC                        whatever        2022/3/29 0:... PartnerSupported

PS C:\Software> Add-EsxSoftwarePackage -ImageProfile "ESX-NUC" -SoftwarePackage "net-community"

Name                           Vendor          Last Modified   Acceptance Level
----                           ------          -------------   ----------------
ESX-NUC                        whatever        2022/5/25 23... PartnerSupported

PS C:\Software> Export-ESXImageProfile -ImageProfile "ESX-NUC" -ExportToISO -filepath ESXi-NUC.iso
```

## 制作启动盘并安装 ESXi

使用 [Rufus](https://rufus.ie/zh/) 工具，选择镜像和 U 盘，其它参数默认，最后写入进 U 盘即可。

安装过程和安装官方标准镜像一致，插入 U 盘开机时按 F10 选择启动项即可进入安装界面。
