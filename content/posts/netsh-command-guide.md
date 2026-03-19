+++
title = "Windows netsh 命令完全指南：网络配置与管理实战"
date = "2026-03-18T10:00:00+08:00"

[taxonomies]
tags = ["Windows", "网络", "netsh", "运维", "命令行"]
categories = ["网络"]

[extra]
summary = "全面介绍 Windows netsh 命令的各个功能模块，包括网络接口配置、防火墙管理、WLAN 设置、端口映射等实用技巧，是 Windows 网络管理员的必备参考手册。"
author = "博主"
+++

在 Windows 系统网络管理中，**netsh**（Network Shell）是一个功能强大的命令行工具，它提供了对网络配置的全面控制能力。无论是配置 IP 地址、管理防火墙、设置无线网络，还是进行网络诊断，netsh 都能提供统一的命令行接口。

本文将全面介绍 netsh 的各个功能模块，帮助你掌握这个 Windows 网络管理的利器。

---

## 一、netsh 基础入门

### 什么是 netsh？

**netsh** 是 Windows 系统自带的网络配置命令行工具，从 Windows 2000 开始就内置于系统中。它提供了一个分层级的命令结构，允许用户配置和监控几乎所有的网络组件。

```cmd
netsh 命令结构：
netsh [上下文] [子上下文] [命令] [参数]

示例：
netsh interface ip set address "以太网" static 192.168.1.100 255.255.255.0 192.168.1.1
```

### 运行模式

netsh 支持两种运行模式：

#### 1. 交互式模式

```cmd
C:\> netsh
netsh> interface
netsh interface> ip
netsh interface ip> show config
netsh interface ip> exit
netsh> exit
```

#### 2. 直接命令模式

```cmd
C:\> netsh interface ip show config
C:\> netsh advfirewall show allprofiles
```

### 常用通用命令

```cmd
# 查看所有可用上下文
netsh /?

# 查看特定上下文的帮助
netsh interface /?
netsh interface ip /?

# 查看命令语法
netsh interface ip set address /?

# 执行脚本文件
netsh -f script.txt
netsh -c interface -f interface-config.txt
```

---

## 二、网络接口配置（interface 上下文）

### 查看接口信息

```cmd
# 显示所有网络接口配置
netsh interface show interface

# 显示特定接口的详细配置
netsh interface ip show config "以太网"

# 显示所有接口的 IP 配置
netsh interface ip show addresses

# 显示 DNS 配置
netsh interface ip show dns

# 显示 WINS 配置
netsh interface ip show wins

# 显示接口统计信息
netsh interface ip show interfaces
```

### 配置 IPv4 地址

#### 静态 IP 配置

```cmd
# 设置静态 IP 地址、子网掩码和默认网关
netsh interface ip set address "以太网" static 192.168.1.100 255.255.255.0 192.168.1.1

# 设置静态 IP 并指定跃点数
netsh interface ip set address "以太网" static 192.168.1.100 255.255.255.0 192.168.1.1 1

# 添加额外的 IP 地址
netsh interface ip add address "以太网" 192.168.1.101 255.255.255.0
```

#### DHCP 自动获取

```cmd
# 启用 DHCP 自动获取 IP
netsh interface ip set address "以太网" dhcp

# 释放并重新获取 IP（配合 ipconfig）
ipconfig /release
ipconfig /renew
```

### 配置 DNS 服务器

```cmd
# 设置主 DNS 服务器
netsh interface ip set dns "以太网" static 8.8.8.8

# 设置主 DNS 并指定跃点数
netsh interface ip set dns "以太网" static 8.8.8.8 primary

# 添加备用 DNS 服务器
netsh interface ip add dns "以太网" 8.8.4.4 index=2

# 启用自动获取 DNS
netsh interface ip set dns "以太网" dhcp

# 清除所有 DNS 服务器
netsh interface ip delete dns "以太网" all
```

### 配置 IPv6

```cmd
# 设置静态 IPv6 地址
netsh interface ipv6 set address "以太网" static 2001:db8::100 64

# 添加 IPv6 默认网关
netsh interface ipv6 add route ::/0 "以太网" fe80::1

# 启用 IPv6 自动配置
netsh interface ipv6 set address "以太网" dhcp

# 禁用 IPv6
netsh interface ipv6 set interface "以太网" disabled
```

### 接口启用/禁用

```cmd
# 禁用网络接口
netsh interface set interface "以太网" disable

# 启用网络接口
netsh interface set interface "以太网" enable

# 重命名接口
netsh interface set interface name="本地连接" newname="以太网"
```

### 备份和恢复配置

```cmd
# 导出接口配置
netsh interface dump > interface-config.txt

# 导入接口配置
netsh -f interface-config.txt

# 导出完整网络配置
netsh dump > full-network-config.txt
```

---

## 三、Windows 防火墙管理（advfirewall 上下文）

Windows Defender 防火墙是 netsh 最常用功能之一，advfirewall 上下文提供了完整的防火墙管理能力。

### 查看防火墙状态

```cmd
# 显示所有配置文件状态
netsh advfirewall show allprofiles

# 显示特定配置文件状态
netsh advfirewall show currentprofile
netsh advfirewall show domainprofile
netsh advfirewall show privateprofile
netsh advfirewall show publicprofile

# 查看防火墙状态（简化）
netsh advfirewall show state
```

### 配置防火墙全局设置

```cmd
# 启用防火墙
netsh advfirewall set allprofiles state on
netsh advfirewall set domainprofile state on
netsh advfirewall set privateprofile state on
netsh advfirewall set publicprofile state on

# 禁用防火墙
netsh advfirewall set allprofiles state off

# 设置默认入站策略（阻止/允许）
netsh advfirewall set allprofiles firewallpolicy blockinbound,allowoutbound
netsh advfirewall set allprofiles firewallpolicy allowinbound,allowoutbound

# 设置日志配置
netsh advfirewall set allprofiles logging filename %systemroot%\system32\LogFiles\Firewall\pfirewall.log
netsh advfirewall set allprofiles logging maxfilesize 4096
netsh advfirewall set allprofiles logging droppedconnections enable
netsh advfirewall set allprofiles logging allowedconnections enable
```

### 管理入站/出站规则

#### 创建入站规则

```cmd
# 允许特定端口入站
netsh advfirewall firewall add rule name="允许 HTTP" dir=in action=allow protocol=TCP localport=80

# 允许特定程序入站
netsh advfirewall firewall add rule name="允许 Chrome" dir=in action=allow program="C:\Program Files\Google\Chrome\Application\chrome.exe" enable=yes

# 允许特定服务入站
netsh advfirewall firewall add rule name="允许远程桌面" dir=in action=allow service=remotedesktop enable=yes

# 允许 ICMP（Ping）
netsh advfirewall firewall add rule name="允许 Ping" dir=in action=allow protocol=icmpv4:any,any

# 允许 IP 范围访问特定端口
netsh advfirewall firewall add rule name="允许内网 SSH" dir=in action=allow protocol=TCP localport=22 remoteip=192.168.1.0/24

# 允许特定应用和服务
netsh advfirewall firewall add rule name="允许 Web 服务器" dir=in action=allow protocol=TCP localport=80,443,8080
```

#### 创建出站规则

```cmd
# 阻止特定程序出站
netsh advfirewall firewall add rule name="阻止程序联网" dir=out action=block program="C:\path\to\app.exe" enable=yes

# 阻止访问特定端口
netsh advfirewall firewall add rule name="阻止 FTP" dir=out action=block protocol=TCP remoteport=21

# 阻止访问特定 IP
netsh advfirewall firewall add rule name="阻止恶意 IP" dir=out action=block remoteip=10.0.0.100
```

#### 查看和管理规则

```cmd
# 查看所有规则
netsh advfirewall firewall show rule name=all

# 查看特定名称的规则
netsh advfirewall firewall show rule name="允许 HTTP"

# 查看特定规则的详细信息
netsh advfirewall firewall show rule name="允许 HTTP" verbose

# 查看所有入站规则（使用 PowerShell 过滤）
powershell -c "Get-NetFirewallRule -Direction Inbound | Where-Object {$_.Enabled -eq True}"

# 查看所有出站规则（使用 PowerShell 过滤）
powershell -c "Get-NetFirewallRule -Direction Outbound | Where-Object {$_.Enabled -eq True}"

# 删除规则
netsh advfirewall firewall delete rule name="允许 HTTP"
netsh advfirewall firewall delete rule name="允许 HTTP" protocol=tcp localport=80

# 禁用/启用规则
netsh advfirewall firewall set rule name="允许 HTTP" new enable=no
netsh advfirewall firewall set rule name="允许 HTTP" new enable=yes

# 修改规则
netsh advfirewall firewall set rule name="允许 HTTP" new localport=8080
```

### 配置安全连接规则

```cmd
# 创建 IPsec 规则
netsh advfirewall consec add rule name="要求加密" endpoint1=any endpoint2=any auth1=computerpsk auth1psk=MySecretKey123 qmsecmethods=ESP[AES256,SHA256]

# 查看连接安全规则
netsh advfirewall consec show rule name=all
```

### 配置受保护的端口

```cmd
# 添加受保护的端口（需要管理员权限）
netsh advfirewall set protectedport 445 protocol=tcp enabled=yes

# 查看受保护的端口
netsh advfirewall show protectedport
```

---

## 四、无线网络管理（wlan 上下文）

netsh wlan 提供了完整的无线网络管理功能，包括扫描、连接、配置文件管理等。

### 查看无线适配器信息

```cmd
# 显示无线适配器状态
netsh wlan show interfaces

# 显示无线网络驱动程序信息
netsh wlan show drivers

# 显示无线网络能力
netsh wlan show wirelesscapabilities

# 显示所有无线配置文件
netsh wlan show profiles

# 显示特定配置文件详情
netsh wlan show profile name="MyWiFi"
netsh wlan show profile name="MyWiFi" key=clear
```

### 扫描和连接网络

```cmd
# 扫描可用无线网络
netsh wlan show networks

# 扫描并显示详细信息
netsh wlan show networks mode=bssid

# 连接到网络（使用已有配置文件）
netsh wlan connect name="MyWiFi"

# 连接到特定 SSID
netsh wlan connect name="MyWiFi" ssid="MyWiFi"

# 断开连接
netsh wlan disconnect

# 阻止网络
netsh wlan block network name="UnwantedNetwork"

# 允许网络
netsh wlan allow network name="MyWiFi"
```

### 管理无线配置文件

```cmd
# 导出配置文件
netsh wlan export profile name="MyWiFi" folder="C:\profiles" key=clear

# 导入配置文件
netsh wlan add profile filename="C:\profiles\WiFi-MyWiFi.xml"

# 删除配置文件
netsh wlan delete profile name="MyWiFi"

# 重命名配置文件
netsh wlan rename profile name="OldName" newname="NewName"

# 设置配置文件优先级
netsh wlan set profileorder name="MyWiFi" interface="Wi-Fi" priority=1
```

### 创建无线配置文件

```cmd
# 创建 WPA2-Personal 配置文件
netsh wlan add profile filename="profile.xml"

# profile.xml 内容示例：
<?xml version="1.0"?>
<netshWlanProfile xmlns="http://www.microsoft.com/networking/WLAN/profile/v1">
    <name>MyWiFi</name>
    <SSIDConfig>
        <SSID>
            <name>MyWiFi</name>
        </SSID>
    </SSIDConfig>
    <connectionType>ESS</connectionType>
    <connectionMode>auto</connectionMode>
    <MSM>
        <security>
            <authEncryption>
                <authentication>WPA2PSK</authentication>
                <encryption>AES</encryption>
                <useOneX>false</useOneX>
            </authEncryption>
            <sharedKey>
                <keyType>passPhrase</keyType>
                <protected>false</protected>
                <keyMaterial>MyPassword123</keyMaterial>
            </sharedKey>
        </security>
    </MSM>
</netshWlanProfile>
```

### WLAN 高级设置

```cmd
# 设置自动连接
netsh wlan set autoconfig enabled=yes interface="Wi-Fi"

# 设置网络列表策略
netsh wlan set allowexplicitcreds enabled=no interface="Wi-Fi"

# 配置托管网络
netsh wlan set hostednetwork mode=allow ssid="MyHotspot" key=MyPassword123
netsh wlan start hostednetwork
netsh wlan stop hostednetwork
```

---

## 五、端口映射和 NAT（portproxy 上下文）

netsh portproxy 用于配置 IPv4 到 IPv4 的端口映射，常用于端口转发场景。

### 查看端口代理配置

```cmd
# 显示所有端口映射
netsh interface portproxy show all

# 显示 IPv4 到 IPv4 的映射
netsh interface portproxy show v4tov4

# 显示 IPv6 到 IPv4 的映射
netsh interface portproxy show v6tov4

# 显示 IPv4 到 IPv6 的映射
netsh interface portproxy show v4tov6

# 显示 IPv6 到 IPv6 的映射
netsh interface portproxy show v6tov6
```

### 配置端口转发

```cmd
# 添加端口映射（本地端口转发）
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=127.0.0.1

# 添加端口映射（转发到其他主机）
netsh interface portproxy add v4tov4 listenport=3389 listenaddress=0.0.0.0 connectport=3389 connectaddress=192.168.1.100

# 添加多个端口范围
netsh interface portproxy add v4tov4 listenport=8000-8010 listenaddress=0.0.0.0 connectport=8000-8010 connectaddress=192.168.1.100

# 删除端口映射
netsh interface portproxy delete v4tov4 listenport=8080 listenaddress=0.0.0.0

# 删除所有映射
netsh interface portproxy delete all
```

### 完整端口转发示例

```cmd
# 场景：将外部访问的 80 端口转发到内网 Web 服务器
netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=192.168.1.100

# 场景：RDP 远程桌面转发
netsh interface portproxy add v4tov4 listenport=33890 listenaddress=0.0.0.0 connectport=3389 connectaddress=192.168.1.101

# 场景：数据库端口转发
netsh interface portproxy add v4tov4 listenport=3306 listenaddress=127.0.0.1 connectport=3306 connectaddress=192.168.1.100

# 验证配置
netsh interface portproxy show all

# 确保防火墙允许相应端口
netsh advfirewall firewall add rule name="端口转发 80" dir=in action=allow protocol=TCP localport=80
```

---

## 六、HTTP 代理配置（http 上下文）

netsh http 用于配置 HTTP Server API 和 URL 保留字，常用于 IIS 和自托管 Web 服务。

### 查看 HTTP 配置

```cmd
# 显示 URL 保留字
netsh http show urlacl

# 显示 SSL 证书绑定
netsh http show sslcert

# 显示 IP 监听列表
netsh http show iplisten

# 显示服务配置
netsh http show service
```

### 配置 URL 保留字

```cmd
# 添加 URL 保留字（允许非管理员程序绑定端口）
netsh http add urlacl url=http://+:8080/ user=Everyone

# 添加特定用户的 URL 保留字
netsh http add urlacl url=http://localhost:5000/ user=DOMAIN\username

# 删除 URL 保留字
netsh http delete urlacl url=http://+:8080/
```

### 配置 SSL 证书绑定

```cmd
# 添加 SSL 证书绑定
netsh http add sslcert ipport=0.0.0.0:443 certhash=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX appid={00112233-4455-6677-8899-AABBCCDDEEFF}

# 删除 SSL 证书绑定
netsh http delete sslcert ipport=0.0.0.0:443

# 查看证书绑定
netsh http show sslcert
```

### 配置 IP 监听

```cmd
# 添加 IP 监听
netsh http add iplisten ipaddress=192.168.1.100

# 删除 IP 监听
netsh http delete iplisten ipaddress=192.168.1.100

# 清除所有 IP 监听
netsh http delete iplisten
```

---

## 七、网络诊断和监控（diagnostics 上下文）

### 网络连通性测试

```cmd
# 测试网络连接
netsh diagnostics ping adapter="以太网"

# 测试网关连接
netsh diagnostics ping gateway adapter="以太网"

# 测试 DNS 解析
netsh diagnostics ping dns
```

### 网络跟踪和捕获

```cmd
# 启动网络跟踪（需要管理员权限）
netsh trace start capture=yes tracefile=C:\trace.etl

# 启动跟踪并过滤
netsh trace start capture=yes report=disabled tracefile=C:\trace.etl IPv4.Address=192.168.1.1

# 停止跟踪
netsh trace stop

# 显示跟踪状态
netsh trace show status

# 转换跟踪文件格式
netsh trace convert input=C:\trace.etl output=C:\trace.cab report=disabled
```

### 网络状态监控

```cmd
# 显示网络统计信息
netsh interface show interface

# 显示接口统计
netsh interface ip show interfaces

# 显示 TCP 统计
netsh interface ip show tcpstats

# 显示 UDP 统计
netsh interface ip show udpstats

# 显示 ICMP 统计
netsh interface ip show icmpstats
```

---

## 八、路由配置（routing 上下文）

### 查看路由表

```cmd
# 显示 IPv4 路由表
netsh interface ipv4 show routes

# 显示 IPv6 路由表
netsh interface ipv6 show routes

# 显示持久路由
netsh interface ipv4 show persistentroutes
```

### 添加和删除路由

```cmd
# 添加默认网关
netsh interface ipv4 add route 0.0.0.0/0 "以太网" 192.168.1.1

# 添加静态路由
netsh interface ipv4 add route 10.0.0.0/8 "以太网" 192.168.1.1

# 添加持久路由（重启后保留）
netsh interface ipv4 add route 10.0.0.0/8 "以太网" 192.168.1.1 store=persistent

# 添加 IPv6 路由
netsh interface ipv6 add route 2001:db8::/32 "以太网" fe80::1

# 删除路由
netsh interface ipv4 delete route 10.0.0.0/8 "以太网"

# 删除所有持久路由
netsh interface ipv4 delete persistentroutes
```

### 路由优先级配置

```cmd
# 设置接口跃点数
netsh interface ipv4 set interface "以太网" metric=10

# 查看接口跃点数
netsh interface ipv4 show interfaces
```

---

## 九、DHCP 和 DNS 管理

### DHCP 客户端配置

```cmd
# 刷新 DHCP 租约
netsh interface ip set address "以太网" dhcp

# 刷新 DNS
netsh interface ip set dns "以太网" dhcp

# 显示 DHCP 类别
netsh interface ip show dhcpclass
```

### DNS 缓存管理

```cmd
# 显示 DNS 缓存
ipconfig /displaydns

# 清除 DNS 缓存
ipconfig /flushdns

# 注册 DNS
ipconfig /registerdns
```

### 配置 DNS 后缀

```cmd
# 设置 DNS 后缀
netsh interface ip set dns "以太网" static 8.8.8.8
netsh interface ip add dns "以太网" 8.8.4.4 index=2

# 配置 DNS 后缀搜索列表
netsh interface ip set dnssuffixes "以太网" static example.com corp.example.com
```

---

## 十、高级脚本和自动化

### 批处理脚本示例

#### 网络配置切换脚本

```batch
@echo off
echo 正在切换网络配置...

if "%1"=="home" (
    echo 应用家庭网络配置
    netsh interface ip set address "以太网" dhcp
    netsh interface ip set dns "以太网" dhcp
) else if "%1"=="work" (
    echo 应用工作网络配置
    netsh interface ip set address "以太网" static 10.0.0.100 255.255.255.0 10.0.0.1
    netsh interface ip set dns "以太网" static 10.0.0.10
    netsh interface ip add dns "以太网" 10.0.0.11 index=2
) else (
    echo 用法：%0 [home^|work]
    exit /b 1
)

echo 配置完成！
netsh interface ip show config "以太网"
```

#### 防火墙规则备份脚本

```batch
@echo off
set BACKUP_DIR=C:\FirewallBackups
set DATE=%date:~0,4%%date:~5,2%%date:~8,2%_%time:~0,2%%time:~3,2%
set DATE=%DATE: =0%

if not exist %BACKUP_DIR% mkdir %BACKUP_DIR%

echo 正在备份防火墙规则...
netsh advfirewall export "%BACKUP_DIR%\firewall-%DATE%.wfw"

echo 备份完成：%BACKUP_DIR%\firewall-%DATE%.wfw
```

#### 一键网络重置脚本

```batch
@echo off
echo 警告：此操作将重置所有网络配置！
pause

echo 重置 Winsock...
netsh winsock reset

echo 重置 TCP/IP...
netsh int ip reset

echo 重置 WinHTTP 代理...
netsh winhttp reset proxy

echo 刷新 DNS...
ipconfig /flushdns

echo 重置完成！请重启计算机。
```

### PowerShell 集成

```cmd
# 在 PowerShell 中使用 netsh
netsh interface ip show config

# 将输出转换为对象
$interfaces = netsh interface show interface | Select-Object -Skip 4
$interfaces

# 批量配置
$profiles = @("Profile1", "Profile2", "Profile3")
foreach ($profile in $profiles) {
    netsh wlan delete profile name="$profile"
}
```

---

## 十一、常见问题排查

### 问题 1：无法修改网络配置

**解决方案**：以管理员身份运行命令提示符

```cmd
# 右键点击 CMD -> 以管理员身份运行
# 或使用 PowerShell 管理员模式
Start-Process cmd -Verb RunAs
```

### 问题 2：防火墙规则不生效

**排查步骤**：

```cmd
# 检查防火墙状态
netsh advfirewall show allprofiles

# 检查规则是否启用
netsh advfirewall firewall show rule name=all | findstr /i "规则名称"

# 检查规则优先级（数字越小优先级越高）
netsh advfirewall firewall show rule name=all verbose
```

### 问题 3：无线网络无法连接

**排查步骤**：

```cmd
# 检查无线适配器状态
netsh wlan show interfaces

# 查看可用网络
netsh wlan show networks mode=bssid

# 删除并重新创建配置文件
netsh wlan delete profile name="MyWiFi"
netsh wlan add profile filename="profile.xml"

# 检查驱动程序
netsh wlan show drivers
```

### 问题 4：端口转发不工作

**排查步骤**：

```cmd
# 检查端口代理配置
netsh interface portproxy show all

# 检查防火墙规则
netsh advfirewall firewall show rule name=all | findstr /i "端口号"

# 检查 Routing and Remote Access 服务
sc query RemoteAccess

# 确保 IP 转发已启用
reg query HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters /v IpEnableRouter
```

### 问题 5：网络配置丢失

**恢复方法**：

```cmd
# 从备份恢复
netsh -f interface-config.txt

# 恢复防火墙配置
netsh advfirewall import "C:\backup\firewall.wfw"

# 重置网络组件
netsh winsock reset
netsh int ip reset
```

---

## 十二、最佳实践和安全建议

### 1. 配置管理最佳实践

```cmd
# 定期备份配置
netsh advfirewall export "C:\Backups\firewall-%date%.wfw"
netsh interface dump > "C:\Backups\interface-%date%.txt"

# 使用版本控制管理脚本
# 将 netsh 脚本纳入 Git 管理

# 文档化所有自定义规则
# 在脚本中添加详细注释说明规则用途
```

### 2. 安全加固建议

```cmd
# 禁用不必要的网络接口
netsh interface set interface "本地连接 2" disable

# 设置严格的防火墙默认策略
netsh advfirewall set allprofiles firewallpolicy blockinbound,allowoutbound

# 启用防火墙日志
netsh advfirewall set allprofiles logging droppedconnections enable

# 定期审查规则
netsh advfirewall firewall show rule name=all verbose
```

### 3. 性能优化

```cmd
# 清理无用的防火墙规则
netsh advfirewall firewall delete rule name="过时规则"

# 优化接口跃点数
netsh interface ipv4 set interface "以太网" metric=10

# 禁用 IPv6（如不需要）
netsh interface ipv6 set interface "以太网" disabled
```

---

## 十三、常用命令速查表

### 接口配置

| 命令 | 功能 |
|------|------|
| `netsh interface ip show config` | 显示接口配置 |
| `netsh interface ip set address "名称" static IP 掩码 网关` | 设置静态 IP |
| `netsh interface ip set address "名称" dhcp` | 启用 DHCP |
| `netsh interface ip set dns "名称" static DNS` | 设置 DNS |

### 防火墙管理

| 命令 | 功能 |
|------|------|
| `netsh advfirewall show allprofiles` | 显示防火墙状态 |
| `netsh advfirewall firewall add rule name="名称" dir=in action=allow protocol=TCP localport=端口` | 添加入站规则 |
| `netsh advfirewall firewall delete rule name="名称"` | 删除规则 |
| `netsh advfirewall export backup.wfw` | 导出配置 |

### 无线网络

| 命令 | 功能 |
|------|------|
| `netsh wlan show interfaces` | 显示无线接口 |
| `netsh wlan show networks` | 扫描网络 |
| `netsh wlan connect name="配置文件名"` | 连接网络 |
| `netsh wlan show profile name="名称" key=clear` | 显示 WiFi 密码 |

### 端口转发

| 命令 | 功能 |
|------|------|
| `netsh interface portproxy show all` | 显示端口映射 |
| `netsh interface portproxy add v4tov4 listenport=端口 connectport=端口 connectaddress=IP` | 添加映射 |
| `netsh interface portproxy delete v4tov4 listenport=端口` | 删除映射 |

---

## 总结

netsh 是 Windows 网络管理的强大工具，涵盖了从基础网络配置到高级防火墙管理的各个方面。掌握 netsh 可以帮助你：

- **快速配置网络**：无需进入图形界面，命令行一键配置
- **批量部署**：通过脚本在多台计算机上统一部署网络配置
- **故障排查**：深入了解网络配置细节，快速定位问题
- **自动化运维**：将网络配置纳入自动化运维流程

建议在实际环境中多加练习，熟悉各个上下文的功能和命令语法。记住，在进行任何网络配置更改之前，务必备份当前配置，以便在出现问题时能够快速恢复。

随着 Windows 系统的发展，部分 netsh 功能可能已被 PowerShell cmdlet 取代（如 NetSecurity 模块），但 netsh 仍然是 Windows 网络管理员必备的核心工具之一。
