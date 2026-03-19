+++
title = "Linux 防火墙完全指南：iptables、nftables 和 firewalld 实战"
date = "2026-03-16T20:12:37+08:00"

[taxonomies]
tags = ["Linux", "防火墙", "安全", "网络", "运维"]
categories = ["操作系统", "网络"]

[extra]
summary = "全面介绍 Linux 三大防火墙工具：iptables、nftables 和 firewalld，包含基础概念、实战配置和最佳实践，帮助你构建安全的网络防护体系。"
author = "博主"
+++

Linux 防火墙是系统安全的第一道防线，对于服务器管理员和运维工程师来说，掌握防火墙配置是必备技能。本文将全面介绍 Linux 系统中三大主流防火墙工具：**iptables**（传统经典）、**nftables**（新一代框架）和 **firewalld**（动态管理），帮助你根据实际需求选择合适的工具，构建安全可靠的网络防护体系。

无论你是刚接触 Linux 的新手，还是有一定经验的管理员，这篇文章都将为你提供从基础概念到高级配置的完整知识体系。

---

## 一、Linux 防火墙基础概念

### 什么是 Netfilter 框架？

在深入了解具体工具之前，我们需要先理解 **Netfilter** —— Linux 内核中的包过滤框架。所有 Linux 防火墙工具（iptables、nftables、firewalld）本质上都是基于 Netfilter 框架的用户态接口。

```
用户空间                    内核空间
┌─────────┐              ┌─────────────┐
│iptables │─────────────>│             │
│nftables │─────────────>│  Netfilter  │────> 网络接口
│firewalld│─────────────>│   框架       │
└─────────┘              └─────────────┘
```

### 核心概念

#### 1. 表（Tables）
表是规则的容器，按功能分类：

| 表名 | 功能 | 常用链 |
|------|------|--------|
| **filter** | 包过滤（默认） | INPUT、FORWARD、OUTPUT |
| **nat** | 网络地址转换 | PREROUTING、POSTROUTING、OUTPUT |
| **mangle** | 修改包头部 | 所有链 |
| **raw** | 连接追踪豁免 | PREROUTING、OUTPUT |

#### 2. 链（Chains）
链是规则的执行顺序，代表数据包的不同处理阶段：

```
                    ┌─────────────┐
    数据包进入 ─────> │ PREROUTING  │────┐
                    └─────────────┘    │
                           │           │
                    ┌──────┴──────┐    │
                    │   路由决策   │    │
                    └──────┬──────┘    │
                           │           │
              ┌────────────┼────────────┤
              │            │            │
              ▼            ▼            ▼
        ┌─────────┐  ┌──────────┐  ┌──────────┐
        │  INPUT  │  │ FORWARD  │  │  OUTPUT  │
        │(本机进程)│  │(转发包)   │  │(本机发出) │
        └────┬────┘  └────┬─────┘  └────┬─────┘
             │            │             │
             ▼            ▼             ▼
        ┌─────────┐  ┌──────────────┐  ┌─────────────┐
        │ 本机进程 │  │  POSTROUTING │  │ POSTROUTING │
        │         │  │(修改源地址)    │  │(修改源地址)  │
        └─────────┘  └──────────────┘  └─────────────┘
```

#### 3. 规则（Rules）
规则由匹配条件和目标动作组成：
- **匹配条件**：源/目标 IP、端口、协议、接口等
- **目标动作**：ACCEPT、DROP、REJECT、LOG、NAT 等

#### 4. 默认策略（Policy）
当没有规则匹配时执行的动作：
- **ACCEPT**：允许通过
- **DROP**：静默丢弃（推荐）
- **REJECT**：拒绝并返回错误信息

---

## 二、iptables：经典防火墙工具

iptables 是 Linux 最经典的防火墙工具，虽然逐渐被 nftables 取代，但在许多生产环境中仍在广泛使用。

### 基本命令结构

```bash
iptables [-t 表名] 命令 [链名] [匹配条件] [-j 目标动作]
```

### 常用命令

#### 查看规则

```bash
# 查看所有规则（详细）
iptables -L -n -v

# 查看特定表
iptables -t nat -L -n -v

# 查看规则编号（方便删除）
iptables -L --line-numbers

# 查看原始规则（可用于备份）
iptables-save
```

#### 添加规则

```bash
# 在链末尾添加规则（-A）
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 在链开头插入规则（-I，优先级更高）
iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT

# 替换指定位置的规则
iptables -R INPUT 1 -p tcp --dport 443 -j ACCEPT
```

#### 删除规则

```bash
# 按规则内容删除
iptables -D INPUT -p tcp --dport 80 -j ACCEPT

# 按规则编号删除
iptables -D INPUT 2

# 清空所有规则
iptables -F
iptables -X  # 删除自定义链
iptables -Z  # 清空计数器
```

### 实战配置示例

#### 场景 1：Web 服务器基础防护

```bash
#!/bin/bash

# 清空现有规则
iptables -F
iptables -X

# 设置默认策略（拒绝所有入站，允许所有出站）
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# 允许本地回环
iptables -A INPUT -i lo -j ACCEPT

# 允许已建立的连接
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 允许 SSH（必须！防止把自己锁在外面）
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 允许 HTTP 和 HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 允许 Ping（可选）
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# 记录并丢弃其他入站连接
iptables -A INPUT -j LOG --log-prefix "IPTABLES DROP: "
iptables -A INPUT -j DROP

echo "防火墙规则已应用"
```

#### 场景 2：限制连接速率（防 DDoS）

```bash
# 限制 SSH 连接速率（每分钟最多 3 个新连接）
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP

# 限制 HTTP 请求速率
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT

# 防止 SYN 洪水攻击
iptables -A INPUT -p tcp --syn -m limit --limit 1/second --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

#### 场景 3：端口转发和 NAT

```bash
# 开启 IP 转发
echo 1 > /proc/sys/net/ipv4/ip_forward

# SNAT（源地址转换，内网访问外网）
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# DNAT（目标地址转换，外网访问内网）
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80
iptables -A FORWARD -p tcp -d 192.168.1.100 --dport 80 -j ACCEPT

# 端口重定向（本机）
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

#### 场景 4：IP 黑白名单

```bash
# 黑名单：阻止特定 IP
iptables -A INPUT -s 192.168.1.100 -j DROP
iptables -A INPUT -s 10.0.0.0/24 -j DROP

# 白名单：只允许特定 IP 访问 SSH
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# 阻止特定国家的 IP（需要 geoip 模块）
iptables -A INPUT -m geoip --src-cc CN -j DROP
```

### 保存和恢复规则

```bash
# Debian/Ubuntu
iptables-save > /etc/iptables/rules.v4
iptables-restore < /etc/iptables/rules.v4

# CentOS/RHEL
service iptables save

# 使用 iptables-persistent（Debian/Ubuntu）
apt-get install iptables-persistent
netfilter-persistent save
```

---

## 三、nftables：新一代防火墙框架

nftables 是 iptables 的继任者，从 Linux 3.13 开始引入，提供了更简洁的语法和更好的性能。

### nftables vs iptables

| 特性 | iptables | nftables |
|------|----------|----------|
| 语法复杂度 | 复杂，多工具 | 统一，简洁 |
| 性能 | 一般 | 更好 |
| 增量更新 | 困难 | 原子操作 |
| IPv4/IPv6 | 分开管理 | 统一处理 |
| 调试 | 困难 | 内置跟踪 |

### 基本命令结构

```bash
# nftables 使用 nft 命令
nft [选项] [命令]

# 常用命令
nft list ruleset          # 查看所有规则
nft add rule ...          # 添加规则
nft delete rule ...       # 删除规则
nft flush ruleset         # 清空所有规则
```

### 实战配置示例

#### 基础配置脚本

```bash
#!/bin/bash

# 清空现有规则
nft flush ruleset

# 创建表（inet 同时支持 IPv4 和 IPv6）
nft add table inet filter

# 创建链
nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
nft add chain inet filter forward { type filter hook forward priority 0 \; policy drop \; }
nft add chain inet filter output { type filter hook output priority 0 \; policy accept \; }

# 允许本地回环
nft add rule inet filter input iif lo accept

# 允许已建立的连接
nft add rule inet filter input ct state established,related accept

# 允许 SSH
nft add rule inet filter input tcp dport 22 accept

# 允许 HTTP/HTTPS
nft add rule inet filter input tcp dport { 80, 443 } accept

# 允许 Ping
nft add rule inet filter input icmp type echo-request accept
nft add rule inet filter input icmpv6 type echo-request accept

echo "nftables 规则已应用"
```

#### 高级配置示例

```bash
# 创建自定义链
nft add chain inet filter tcp_chain
nft add chain inet filter udp_chain

# 按协议分流
nft add rule inet filter input ip protocol tcp jump tcp_chain
nft add rule inet filter input ip protocol udp jump udp_chain

# 在自定义链中添加规则
nft add rule inet filter tcp_chain tcp dport 22 accept
nft add rule inet filter tcp_chain tcp dport 80 accept

# 使用集合（Set）管理端口
nft add set inet filter allowed_ports { type inet_service \; flags interval \; }
nft add element inet filter allowed_ports { 22, 80, 443, 8080-8090 }
nft add rule inet filter input tcp dport @allowed_ports accept

# 使用映射（Map）进行动态处理
nft add map inet filter port_to_action { type inet_service : verdict \; }
nft add element inet filter port_to_action { 22 : accept, 80 : accept, 443 : accept }
nft add rule inet filter input tcp dport vmap @port_to_action

# 速率限制
nft add rule inet filter input tcp dport 22 limit rate 3/minute accept
nft add rule inet filter input tcp dport 22 drop
```

#### NAT 配置

```bash
# 创建 nat 表
nft add table ip nat

# 创建链
nft add chain ip nat prerouting { type nat hook prerouting priority 0 \; }
nft add chain ip nat postrouting { type nat hook postrouting priority 100 \; }

# SNAT
nft add rule ip nat postrouting oif eth0 masquerade

# DNAT
nft add rule ip nat prerouting tcp dport 8080 dnat to 192.168.1.100:80
```

### 保存和恢复

```bash
# 保存规则
nft list ruleset > /etc/nftables.conf

# 恢复规则
nft -f /etc/nftables.conf

# 设置开机自动加载（systemd）
systemctl enable nftables
```

---

## 四、firewalld：动态防火墙管理

firewalld 是 RHEL/CentOS/Fedora 默认的防火墙管理工具，提供了动态管理功能，无需重启服务即可应用规则变更。

### 核心概念

#### 1. 区域（Zones）
预定义的安全策略集合：

| 区域 | 描述 | 默认规则 |
|------|------|----------|
| **public** | 公共区域 | 仅允许 SSH、DHCP、Ping |
| **home** | 家庭网络 | 允许更多服务 |
| **work** | 工作网络 | 允许工作相关服务 |
| **internal** | 内部网络 | 信任的网络 |
| **dmz** | 隔离区 | 允许有限的外部访问 |
| **block** | 拒绝所有 | 明确拒绝所有入站 |
| **drop** | 静默丢弃 | 静默丢弃所有入站 |

#### 2. 服务（Services）
预定义的服务配置，包含端口和协议信息：

```bash
# 查看可用服务
firewall-cmd --get-services

# 常见服务：ssh, http, https, mysql, postgresql, samba 等
```

### 常用命令

#### 基本操作

```bash
# 查看状态
firewall-cmd --state
firewall-cmd --reload

# 查看默认区域
firewall-cmd --get-default-zone

# 查看当前活动区域
firewall-cmd --get-active-zones

# 查看区域配置
firewall-cmd --zone=public --list-all
```

#### 添加/删除规则

```bash
# 临时添加服务（重启后失效）
firewall-cmd --add-service=http
firewall-cmd --add-service=https

# 永久添加服务
firewall-cmd --permanent --add-service=http
firewall-cmd --reload

# 添加端口
firewall-cmd --add-port=8080/tcp
firewall-cmd --add-port=1000-2000/tcp

# 移除服务/端口
firewall-cmd --remove-service=http
firewall-cmd --remove-port=8080/tcp

# 富规则（Rich Rules）
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept'
```

### 实战配置示例

#### 场景 1：Web 服务器配置

```bash
#!/bin/bash

# 设置默认区域为 public
firewall-cmd --set-default-zone=public

# 允许基本服务
firewall-cmd --permanent --add-service=ssh
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https

# 允许 Ping
firewall-cmd --permanent --add-icmp-block-inversion

# 重新加载
firewall-cmd --reload

# 查看配置
echo "当前配置："
firewall-cmd --list-all
```

#### 场景 2：限制特定 IP 访问

```bash
# 只允许特定 IP 访问 SSH
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept'

# 阻止特定 IP
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.100" reject'

# 速率限制
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" service name="ssh" limit value="3/m" accept'
```

#### 场景 3：端口转发

```bash
# 本地端口转发
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080

# 远程端口转发（DNAT）
firewall-cmd --permanent --add-forward-port=port=8080:proto=tcp:toaddr=192.168.1.100:toport=80

# 开启 IP 伪装（SNAT）
firewall-cmd --permanent --add-masquerade

firewall-cmd --reload
```

#### 场景 4：创建自定义服务

```bash
# 创建自定义服务文件
sudo tee /etc/firewalld/services/myapp.xml << 'EOF'
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>My Application</short>
  <description>My custom application service</description>
  <port protocol="tcp" port="8080"/>
  <port protocol="tcp" port="8443"/>
</service>
EOF

# 重新加载 firewalld
firewall-cmd --reload

# 使用自定义服务
firewall-cmd --permanent --add-service=myapp
firewall-cmd --reload
```

---

## 五、防火墙最佳实践

### 1. 安全原则

```bash
# 原则 1：默认拒绝所有入站
iptables -P INPUT DROP
# 或 nftables: policy drop
# 或 firewalld: 使用 public 区域

# 原则 2：明确允许必要服务
# 只开放业务必需的端口

# 原则 3：最小权限原则
# 限制源 IP 范围
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT

# 原则 4：记录可疑活动
iptables -A INPUT -j LOG --log-prefix "SUSPICIOUS: "
```

### 2. 配置管理

```bash
# 使用版本控制管理防火墙脚本
git add firewall-rules.sh

# 注释清晰
# 允许 SSH 访问 - 2024-01-15 添加
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 测试配置
iptables-restore --test < rules.v4

# 定时备份
0 2 * * * /sbin/iptables-save > /backup/iptables-$(date +\%Y\%m\%d).bak
```

### 3. 监控和日志

```bash
# 查看日志
tail -f /var/log/kern.log | grep IPTABLES

# 统计连接数
conntrack -L | wc -l

# 查看连接状态
ss -tan | awk '{print $1}' | sort | uniq -c

# 使用 fail2ban 自动封禁
apt-get install fail2ban
```

### 4. 故障排查

```bash
# 检查规则是否生效
iptables -L -n -v --line-numbers

# 测试特定端口连通性
nc -zv localhost 80

# 抓包分析
tcpdump -i eth0 port 80

# 检查内核模块
lsmod | grep iptable

# 查看连接追踪表
cat /proc/net/nf_conntrack
```

---

## 六、工具选择建议

| 场景 | 推荐工具 | 理由 |
|------|----------|------|
| 传统服务器维护 | iptables | 兼容性好，文档丰富 |
| 新系统部署 | nftables | 现代、高效、易维护 |
| RHEL/CentOS 桌面 | firewalld | 动态管理，集成度高 |
| 复杂企业环境 | firewalld + nftables | 灵活性和性能兼顾 |
| 嵌入式/容器 | nftables | 轻量，性能优 |

### 迁移建议

```bash
# 从 iptables 迁移到 nftables
# 使用 iptables-translate 工具
iptables-translate -A INPUT -p tcp --dport 22 -j ACCEPT
# 输出：nft add rule ip filter INPUT tcp dport 22 accept

# 完整迁移
iptables-save > rules.iptables
iptables-restore-translate -f rules.iptables > rules.nft
nft -f rules.nft
```

---

## 七、总结

Linux 防火墙是系统安全的基石，选择合适的工具并正确配置至关重要：

- **iptables**：经典稳定，适合传统环境
- **nftables**：现代高效，适合新部署
- **firewalld**：动态管理，适合桌面和简单服务器

无论选择哪种工具，都要遵循**默认拒绝、最小权限、持续监控**的安全原则。定期审查规则、更新策略、分析日志，才能构建真正安全的网络防护体系。

记住：**安全是一个过程，而不是一个产品**。防火墙只是安全体系的一部分，还需要配合其他安全措施（如 SELinux/AppArmor、定期更新、入侵检测等）才能提供全面的保护。
