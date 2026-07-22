---
type: Note
---
# work\\\_notes

## Windows11 开启/关闭SSH

### 开启ssh server

1、检查openssh是否安装

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

2、安装openssh

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

3、启动ssh服务

```
Start-Service sshd
```

1. 设置开机自动启动

```powershell
Set-Service -Name sshd -StartupType Automatic
```

1. 开启 Windows 防火墙 22 端口

先检查是否已有规则：

```powershell
Get-NetFirewallRule -Name sshd
```

如果存在：

```powershell
Enable-NetFirewallRule -Name sshd
```

如果不存在：

```powershell
New-NetFirewallRule `
-Name sshd `
-DisplayName "OpenSSH SSH Server" `
-Enabled True `
-Direction Inbound `
-Protocol TCP `
-Action Allow `
-LocalPort 22
```

1. 检查 SSH 状态

```powershell
Get-Service sshd
```

### 二、关闭 SSH Server

1. 停止 SSH 服务

```powershell
Stop-Service sshd
```

1. 禁止开机启动

```powershell
Set-Service -Name sshd -StartupType Disabled
```

1. 关闭防火墙规则

```powershell
Disable-NetFirewallRule -Name sshd
```

1. （可选）卸载 OpenSSH Server

如果完全不用：

```powershell
Remove-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

## tailscale

### 切换账号

1、命令退出

```powershell
tailscale logout
```

2、重新登录

```powershell
tailscale up
```

3、清理本级配置

```powershell
tailscale reset
```

4、查看当前连接状态

```powershell
tailscale status
```

5、完全卸载tailscale

```powershell
winget uninstall Tailscale.Tailscale
```

## windows开启IP转发

一、Windows 安装并登录 Tailscale

管理员 PowerShell：

1. 查看网卡

```powershell
Get-NetIPInterface
```

找到：

- Tailscale 网卡
- 物理网卡（WiFi/网线）

例如：

```powershell
InterfaceAlias
----------------
Ethernet
Tailscale
```

1. 开启所有网卡转发

执行：

```powershell
Set-NetIPInterface -Forwarding Enabled
```

检查：

```powershell
Get-NetIPInterface | Where-Object {$_.Forwarding -eq "Enabled"}
```

应该看到：

```powershell
Ethernet       Enabled
Tailscale      Enabled
```

三、开启 Windows 路由功能（推荐）

注册表开启：

```powershell
Set-ItemProperty `
-Path "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" `
-Name IPEnableRouter `
-Value 1
```

重启：

```powershell
shutdown /r /t 0
```

四、确认你的内网网段

Windows 执行：

```powershell
ipconfig
```

例如：

```powershell
Ethernet:

IPv4 Address:
172.18.16.100

Subnet Mask:
255.255.255.0
```

说明：

你的内网：

```powershell
172.18.16.0/24
```

五、让 Tailscale 发布这个网段

执行：

```powershell
tailscale up --advertise-routes=172.18.16.0/24
```

例如：

你的 AI 在：

```powershell
172.18.16.23:11434
```

那么发布：

```powershell
tailscale up --advertise-routes=172.18.16.0/24
```

---

如果提示：

```powershell
Warning: updating settings via 'tailscale up' requires specifying all non-default flags
```

使用：

```powershell
tailscale up `
--reset `
--advertise-routes=172.18.16.0/24
```

六、Tailscale 管理后台批准路由

打开：

[Tailscale Admin Console](https://login.tailscale.com/admin/machines?utm_source=chatgpt.com)

找到你的 Windows：

例如：

```powershell
windows-11
```

点击：

```powershell
Edit route settings
```

开启：

```powershell
172.18.16.0/24
```

保存。

七、Mac 开启接受子网路由

Mac：

```powershell
sudo tailscale up --accept-routes
```

检查：

```powershell
tailscale status
```

应该看到：

```powershell
windows-11
subnet routes: 172.18.16.0/24
```

八、测试访问内网资源

1. Ping

Mac：

```powershell
ping 172.18.16.23
```

例如：

```powershell
64 bytes from 172.18.16.23
```

成功。

九、Windows 防火墙放行转发

如果 ping 不通：

管理员 PowerShell：

```powershell
New-NetFirewallRule `
-DisplayName "Tailscale Subnet Router Forward" `
-Direction Inbound `
-Action Allow `
-Protocol Any
```

十、查看 Tailscale 路由状态

Windows：

```powershell
tailscale status
```

查看：

```powershell
tailscale netcheck
```

查看已发布：

```powershell
tailscale debug prefs
```

应该有：

```powershell
AdvertiseRoutes:
[
172.18.16.0/24
]
```

十一、停止子网转发

如果以后不用：

```powershell
tailscale up --reset
```

或者：

```powershell
tailscale set --advertise-routes=
```
