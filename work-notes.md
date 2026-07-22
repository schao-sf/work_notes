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

4. 设置开机自动启动

```powershell
Set-Service -Name sshd -StartupType Automatic
```

5. 开启 Windows 防火墙 22 端口

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

6. 检查 SSH 状态

```powershell
Get-Service sshd
```

### 二、关闭 SSH Server

1. 停止 SSH 服务

```powershell
Stop-Service sshd
```

---

2. 禁止开机启动

```powershell
Set-Service -Name sshd -StartupType Disabled
```

---

3. 关闭防火墙规则

```powershell
Disable-NetFirewallRule -Name sshd
```

---

4. （可选）卸载 OpenSSH Server

如果完全不用：

```powershell
Remove-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

###
