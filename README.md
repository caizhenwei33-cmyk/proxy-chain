# 🔗 代理链 / 多级代理搭建教程

> Proxy Chain & Multi-level Proxy Setup Guide | 从 HTTP 到 SOCKS5，从正向代理到反向代理链

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 📋 目录 | Table of Contents

- [📋 目录 | Table of Contents](#-目录--table-of-contents)
- [一、概述 | Overview](#一概述--overview)
  - [什么是代理链？ | What is a Proxy Chain?](#什么是代理链--what-is-a-proxy-chain)
  - [代理类型对比 | Proxy Type Comparison](#代理类型对比--proxy-type-comparison)
  - [应用场景 | Use Cases](#应用场景--use-cases)
- [二、基础代理搭建 | Basic Proxy Setup](#二基础代理搭建--basic-proxy-setup)
  - [2.1 Squid HTTP/HTTPS 正向代理 | Squid HTTP/HTTPS Forward Proxy](#21-squid-httphttps-正向代理--squid-httphttps-forward-proxy)
  - [2.2 Dante SOCKS5 代理 | Dante SOCKS5 Proxy](#22-dante-socks5-代理--dante-socks5-proxy)
  - [2.3 TinyProxy 轻量级 HTTP 代理 | TinyProxy Lightweight HTTP Proxy](#23-tinyproxy-轻量级-http-代理--tinyproxy-lightweight-http-proxy)
- [三、代理链搭建 | Proxy Chain Setup](#三代理链搭建--proxy-chain-setup)
  - [3.1 链式代理原理 | Chain Proxy Principle](#31-链式代理原理--chain-proxy-principle)
  - [3.2 方案一：Squid 级联 | Option 1: Squid Cascade](#32-方案一squid-级联--option-1-squid-cascade)
  - [3.3 方案二：ProxyChains-NG 客户端链 | Option 2: ProxyChains-NG Client Chain](#33-方案二proxychains-ng-客户端链--option-2-proxychains-ng-client-chain)
  - [3.4 方案三：HAProxy 中转 | Option 3: HAProxy Relay](#34-方案三haproxy-中转--option-3-haproxy-relay)
- [四、Docker 部署 | Docker Deployment](#四docker-部署--docker-deployment)
  - [4.1 Squid Docker 部署 | Squid Docker Deployment](#41-squid-docker-部署--squid-docker-deployment)
  - [4.2 Dante SOCKS5 Docker 部署 | Dante SOCKS5 Docker Deployment](#42-dante-socks5-docker-部署--dante-socks5-docker-deployment)
  - [4.3 多级代理链 Docker Compose | Multi-level Proxy Chain Docker Compose](#43-多级代理链-docker-compose--multi-level-proxy-chain-docker-compose)
- [五、代理链配置示例 | Proxy Chain Configuration Examples](#五代理链配置示例--proxy-chain-configuration-examples)
  - [5.1 二级正向代理链 | Two-level Forward Proxy Chain](#51-二级正向代理链--two-level-forward-proxy-chain)
  - [5.2 HTTP → SOCKS5 混合链 | HTTP → SOCKS5 Mixed Chain](#52-http--socks5-混合链--http--socks5-mixed-chain)
  - [5.3 反向代理链 | Reverse Proxy Chain](#53-反向代理链--reverse-proxy-chain)
- [六、ProxyChains-NG 详解 | ProxyChains-NG Deep Dive](#六proxychains-ng-详解--proxychains-ng-deep-dive)
  - [安装 | Installation](#安装--installation)
  - [配置 | Configuration](#配置--configuration)
  - [使用 | Usage](#使用--usage)
- [七、常用命令速查 | Common Command Reference](#七常用命令速查--common-command-reference)
- [八、高级技巧 | Advanced Tips](#八高级技巧--advanced-tips)
  - [代理链健康检查 | Proxy Chain Health Check](#代理链健康检查--proxy-chain-health-check)
  - [认证与权限 | Authentication and Authorization](#认证与权限--authentication-and-authorization)
  - [负载均衡代理链 | Load-balanced Proxy Chain](#负载均衡代理链--load-balanced-proxy-chain)
  - [数据加密隧道 | Encrypted Tunnels via Proxy](#数据加密隧道--encrypted-tunnels-via-proxy)
- [九、常见问题 | FAQ](#九常见问题--faq)
- [☕ 支持 / Support](#-支持--support)

---

## 一、概述 | Overview

### 什么是代理链？ | What is a Proxy Chain?

**代理链**（Proxy Chain）是指将多个代理服务器串联起来，让流量依次经过每一级代理，最终到达目标服务器。每一级代理只知道上一级和下一级的地址，无法获知完整路径，从而增强隐私保护和访问控制能力。

A **proxy chain** is a series of proxy servers linked together where traffic flows through each proxy sequentially before reaching the destination. Each proxy only knows its predecessor and successor in the chain, making it difficult to trace the full path, thus enhancing privacy and access control.

```
Client → Proxy1 → Proxy2 → Proxy3 → Internet
```

### 代理类型对比 | Proxy Type Comparison

| 类型 | Protocol | 加密 | 速度 | 适用场景 |
|------|----------|------|------|----------|
| HTTP 代理 | HTTP | ❌ 明文 | ⭐⭐⭐ | 网页浏览、API 调用 |
| HTTPS 代理 | HTTP CONNECT + TLS | ✅ 加密 | ⭐⭐⭐ | 安全浏览 |
| SOCKS4 | TCP | ❌ 明文 | ⭐⭐⭐⭐ | 通用 TCP 代理 |
| SOCKS5 | TCP/UDP | ❌ 明文(可选认证) | ⭐⭐⭐⭐ | 通用代理、UDP 支持 |
| Shadowsocks | 自定义协议 | ✅ 加密 | ⭐⭐⭐ | 科学上网 |
| HTTP 反向代理 | HTTP | ❌/✅ | ⭐⭐⭐ | 负载均衡、隐藏后端 |

### 应用场景 | Use Cases

| 场景 | Description |
|------|-------------|
| 隐私保护 | 多级代理隐藏真实 IP，增强匿名性 |
| 突破封锁 | 绕过地理限制或网络防火墙 |
| 负载分摊 | 将流量分散到多个出口节点 |
| 企业审计 | 通过代理链审计和记录员工网络访问 |
| 多层跳板 | 访问隔离的内网资源（堡垒机 → 内网服务器） |

---

## 二、基础代理搭建 | Basic Proxy Setup

### 2.1 Squid HTTP/HTTPS 正向代理 | Squid HTTP/HTTPS Forward Proxy

**安装 Squid:**

```bash
# Debian / Ubuntu
apt update && apt install -y squid apache2-utils

# CentOS / RHEL
dnf install -y squid httpd-tools
```

**配置 `/etc/squid/squid.conf`:**

```conf
# 监听端口
http_port 3128
# 或 HTTPS 代理端口
# http_port 3128 ssl-bump cert=/etc/squid/cert.pem generate-host-certificates=on

# 访问控制
acl localnet src 0.0.0.0/0
acl SSL_ports port 443
acl Safe_ports port 80
acl Safe_ports port 443
acl CONNECT method CONNECT

# 允许访问
http_access allow localnet
http_access deny all

# 基本认证
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Proxy
auth_param basic credentialsttl 2 hours
acl authenticated proxy_auth REQUIRED
http_access allow authenticated

# 性能
cache_mem 64 MB
maximum_object_size_in_memory 512 KB
cache_dir ufs /var/spool/squid 100 16 256
```

**创建认证用户:**

```bash
htpasswd -c /etc/squid/passwd username
htpasswd /etc/squid/passwd another_user
```

**启动服务:**

```bash
systemctl restart squid
systemctl enable squid
systemctl status squid
```

**测试代理:**

```bash
# 使用 curl 测试
curl -x http://username:password@your-server:3128 https://httpbin.org/ip

# 验证出口 IP
curl -x http://username:password@your-server:3128 https://api.ipify.org
```

### 2.2 Dante SOCKS5 代理 | Dante SOCKS5 Proxy

**安装 Dante:**

```bash
apt update && apt install -y dante-server
```

**配置 `/etc/danted.conf`:**

```conf
logoutput: /var/log/danted.log
internal: 0.0.0.0 port = 1080
external: eth0
socksmethod: username
user.privileged: root
user.unprivileged: nobody

# 客户端规则
client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: error
}

# SOCKS 规则
socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    socksmethod: username
    log: error
}
```

**创建用户:**

```bash
useradd -s /sbin/nologin -M socksuser
passwd socksuser
```

**启动服务:**

```bash
systemctl restart danted
systemctl enable danted
```

**测试 SOCKS5 代理:**

```bash
# 使用 curl 测试
curl -x socks5://socksuser:password@your-server:1080 https://httpbin.org/ip

# 使用 ssh 通过 SOCKS5
ssh -o ProxyCommand='nc -X 5 -x socksuser:password@your-server:1080 %h %p' target-server
```

### 2.3 TinyProxy 轻量级 HTTP 代理 | TinyProxy Lightweight HTTP Proxy

```bash
apt install -y tinyproxy

# 配置 /etc/tinyproxy/tinyproxy.conf
cat > /etc/tinyproxy/tinyproxy.conf << 'EOF'
Port 8888
Allow 0.0.0.0/0
Upstream http proxy-chain-server:3128
Timeout 600
DefaultErrorFile "/usr/share/tinyproxy/default.html"
StatFile "/usr/share/tinyproxy/stats.html"
Logfile "/var/log/tinyproxy/tinyproxy.log"
LogLevel Info
PidFile "/var/run/tinyproxy/tinyproxy.pid"
MaxClients 100
MinSpareServers 5
MaxSpareServers 20
StartServers 10
MaxRequestsPerChild 0
ViaProxyName "tinyproxy"
DisableViaHeader yes
XTinyproxy yes
EOF

systemctl restart tinyproxy
```

---

## 三、代理链搭建 | Proxy Chain Setup

### 3.1 链式代理原理 | Chain Proxy Principle

代理链的核心是**级联**（cascade）或**转发**（forward）。每个代理节点在收到客户端请求后，不是直接访问目标，而是将请求转发到下一级代理。

The core concept of a proxy chain is **cascade** or **forwarding**: each proxy node, instead of reaching the target directly, forwards the request to the next proxy in the chain.

```
Chain Architecture:
┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌─────────┐
│ Client  │───►│ Proxy A  │───►│ Proxy B  │───►│ Proxy C  │───►│ Target  │
│ (China) │    │ (HK)     │    │ (US-W)   │    │ (US-E)   │    │ Server  │
└─────────┘    └──────────┘    └──────────┘    └──────────┘    └─────────┘
```

### 3.2 方案一：Squid 级联 | Option 1: Squid Cascade

Squid 原生支持 `cache_peer` 实现级联代理。

Squid natively supports `cache_peer` for cascade proxying.

**节点 A（入口代理，中国服务器）** - `/etc/squid/squid.conf`:

```conf
http_port 3128

# 访问控制
acl all src 0.0.0.0/0
http_access allow all

# 缓存同级节点：转发到下一级代理
cache_peer proxy-b-ip parent 3128 0 no-query no-digest \
    proxy-only login=user_a:pass_a

# 所有请求通过 cache_peer 转发
never_direct allow all
```

**节点 B（中间代理，香港服务器）** - `/etc/squid/squid.conf`:

```conf
http_port 3128

acl all src 0.0.0.0/0
http_access allow all

# 转发到节点 C
cache_peer proxy-c-ip parent 3128 0 no-query no-digest \
    proxy-only login=user_b:pass_b

never_direct allow all
```

**节点 C（出口代理，美国服务器）** - `/etc/squid/squid.conf`:

```conf
http_port 3128

acl all src 0.0.0.0/0
http_access allow all

# 节点 C 直接访问互联网
# 不需要 cache_peer
```

### 3.3 方案二：ProxyChains-NG 客户端链 | Option 2: ProxyChains-NG Client Chain

使用 proxychains-ng 在客户端实现多级代理链，不需要修改代理服务器配置。

Use proxychains-ng on the client side to implement multi-level proxy chains without modifying proxy server configurations.

**安装:**

```bash
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng
./configure --prefix=/usr --sysconfdir=/etc
make -j$(nproc)
make install
make install-config
```

**配置 `/etc/proxychains.conf`:**

```conf
# 严格链模式：严格按顺序经过每个代理
strict_chain

# 安静模式
quiet_mode

# 代理 DNS
proxy_dns

# 远程 DNS 解析
remote_dns_subnet 224
tcp_read_time_out 15000
tcp_connect_time_out 8000

# 代理列表（顺序连接）
[ProxyList]
# 风格: 类型 IP 端口 [用户 密码]
http     proxy-a-ip    3128    user_a    pass_a
socks5   proxy-b-ip    1080    user_b    pass_b
http     proxy-c-ip    3128    user_c    pass_c
```

### 3.4 方案三：HAProxy 中转 | Option 3: HAProxy Relay

使用 HAProxy 作为代理链的中间继电器。

Use HAProxy as an intermediate relay in the proxy chain.

```haproxy
# /etc/haproxy/haproxy.cfg
global
    daemon
    maxconn 4096

defaults
    mode tcp
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

# 前端：接收客户端 SOCKS5 请求
frontend socks-in
    bind *:1080
    default_backend proxy-chain

# 后端：依次经过多级代理
backend proxy-chain
    # 第一级：Squid HTTP 代理
    server squid-proxy proxy-a-ip:3128 check inter 5s
    
    # 可以通过 ACL 实现条件路由
    # 例如根据源 IP 选择不同链路
```

---

## 四、Docker 部署 | Docker Deployment

### 4.1 Squid Docker 部署 | Squid Docker Deployment

```yaml
# docker-compose-squid.yml
version: '3.8'

services:
  squid:
    image: sameersbn/squid:latest
    container_name: squid-proxy
    restart: always
    ports:
      - "3128:3128"
    volumes:
      - ./squid.conf:/etc/squid/squid.conf:ro
      - ./squid_passwd:/etc/squid/passwd:ro
      - ./squid_cache:/var/spool/squid
    environment:
      - TZ=Asia/Shanghai
```

**Quick start:**

```bash
cat > squid.conf << 'EOF'
http_port 3128
acl all src all
http_access allow all
cache_peer proxy-b-ip parent 3128 0 no-query no-digest proxy-only
never_direct allow all
EOF

docker compose -f docker-compose-squid.yml up -d
```

### 4.2 Dante SOCKS5 Docker 部署 | Dante SOCKS5 Docker Deployment

```yaml
# docker-compose-dante.yml
version: '3.8'

services:
  dante:
    image: vimagick/dante
    container_name: dante-socks5
    restart: always
    ports:
      - "1080:1080"
    volumes:
      - ./danted.conf:/etc/danted.conf:ro
    cap_add:
      - NET_ADMIN
    environment:
      - TZ=Asia/Shanghai
```

**danted.conf for Docker:**

```conf
logoutput: stderr
internal: 0.0.0.0 port = 1080
external: eth0
socksmethod: username

client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
}

socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    socksmethod: username
}
```

### 4.3 多级代理链 Docker Compose | Multi-level Proxy Chain Docker Compose

一键部署三级代理链。One-click deploy a 3-tier proxy chain.

```yaml
# docker-compose-chain.yml
version: '3.8'

networks:
  proxy-network:
    driver: bridge

services:
  # 第一级：入口代理（中国）
  proxy-level1:
    image: sameersbn/squid:latest
    container_name: proxy-l1
    ports:
      - "3128:3128"
    volumes:
      - ./level1.conf:/etc/squid/squid.conf:ro
    networks:
      proxy-network:
        aliases:
          - proxy-l1

  # 第二级：中转代理（香港）
  proxy-level2:
    image: sameersbn/squid:latest
    container_name: proxy-l2
    volumes:
      - ./level2.conf:/etc/squid/squid.conf:ro
    networks:
      proxy-network:
        aliases:
          - proxy-l2

  # 第三级：出口代理（美国）
  proxy-level3:
    image: sameersbn/squid:latest
    container_name: proxy-l3
    volumes:
      - ./level3.conf:/etc/squid/squid.conf:ro
    networks:
      proxy-network:
        aliases:
          - proxy-l3
```

**Level 1 config (level1.conf):** → forwards to level2

```conf
http_port 3128
acl all src all
http_access allow all
cache_peer proxy-l2 parent 3128 0 no-query no-digest proxy-only
never_direct allow all
```

**Level 2 config (level2.conf):** → forwards to level3

```conf
http_port 3128
acl all src all
http_access allow all
cache_peer proxy-l3 parent 3128 0 no-query no-digest proxy-only
never_direct allow all
```

**Level 3 config (level3.conf):** → direct internet access

```conf
http_port 3128
acl all src all
http_access allow all
```

---

## 五、代理链配置示例 | Proxy Chain Configuration Examples

### 5.1 二级正向代理链 | Two-level Forward Proxy Chain

```
Client → Squid(HK) → Squid(US) → Internet
```

**Level 1 (HK):** `/etc/squid/squid.conf`

```conf
http_port 3128
acl all src all
http_access allow all

# Forward to US proxy
cache_peer us-proxy-ip parent 3128 0 no-query no-digest \
    proxy-only login=hku:pass_hk

never_direct allow all
```

**Level 2 (US):** `/etc/squid/squid.conf`

```conf
http_port 3128
acl all src all
http_access allow all

# This proxy accesses the internet directly
# Add authentication if needed:
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
acl authenticated proxy_auth REQUIRED
http_access allow authenticated
```

### 5.2 HTTP → SOCKS5 混合链 | HTTP → SOCKS5 Mixed Chain

使用 proxychains 混合不同类型的代理。Mix different proxy types with proxychains.

```conf
# /etc/proxychains.conf
strict_chain
proxy_dns

[ProxyList]
# Step 1: HTTP proxy
http    hk-proxy    3128    user_hk    pass_hk
# Step 2: SOCKS5 proxy
socks5  us-proxy    1080    user_us    pass_us
# Step 3: SOCKS4 proxy
socks4  eu-proxy    1080
```

**Usage:**

```bash
proxychains4 curl https://httpbin.org/ip
proxychains4 wget https://example.com/file.zip
proxychains4 ssh user@target-server
```

### 5.3 反向代理链 | Reverse Proxy Chain

使用 Nginx 构建反向代理链。Build a reverse proxy chain with Nginx.

```nginx
# 入口 Nginx（公网）
server {
    listen 443 ssl;
    server_name entry.example.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    location / {
        # 转发到内网二级代理
        proxy_pass http://internal-proxy:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```nginx
# 内网二级 Nginx
server {
    listen 8080;

    location / {
        # 转发到实际后端服务
        proxy_pass http://backend-service:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 六、ProxyChains-NG 详解 | ProxyChains-NG Deep Dive

### 安装 | Installation

```bash
# Debian / Ubuntu
apt install -y proxychains4

# 或者编译安装
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng
./configure --prefix=/usr --sysconfdir=/etc
make -j$(nproc)
make install
make install-config
```

### 配置 | Configuration

完整配置文件 `/etc/proxychains.conf` 选项说明：

| 选项 | Description | 默认值 |
|------|-------------|--------|
| `strict_chain` | 严格按顺序连接，一个失败则整个失败 | off |
| `dynamic_chain` | 动态链，跳过不可用的代理 | off |
| `random_chain` | 随机选择代理列表中的代理 | off |
| `chain_len` | 随机链的代理数量（配合 random_chain） | 2 |
| `proxy_dns` | 通过代理解析 DNS | off |
| `remote_dns_subnet` | 远程 DNS 子网 | 224 |
| `tcp_read_time_out` | TCP 读取超时（ms） | 15000 |
| `tcp_connect_time_out` | TCP 连接超时（ms） | 8000 |

**链模式对比:**

```
strict_chain:  A ──► B ──► C          (B 挂了 → 失败)
                                      
dynamic_chain: A ──► B ──► C          (B 挂了 → A ──► C 可用)
                                      
random_chain:  [A, B, C] → 随机选 N 个 (N = chain_len)
```

### 使用 | Usage

```bash
# 基本用法
proxychains4 curl https://api.ipify.org

# 强制 IPv4
proxychains4 -f curl -4 https://api.ipify.org

# 清除 DNS 缓存
proxychains4 -q curl https://example.com

# 指定配置文件
proxychains4 -f /path/to/custom.conf curl https://example.com

# SSH 通过代理链
proxychains4 ssh user@target-server

# git 通过代理链
proxychains4 git clone https://github.com/user/repo.git

# Python 包安装通过代理链
proxychains4 pip install some-package

# 创建别名以简化使用
alias pc='proxychains4'
pc curl https://httpbin.org/ip
```

---

## 七、常用命令速查 | Common Command Reference

| 命令 | Description |
|------|-------------|
| `curl -x http://user:pass@host:3128 https://ifconfig.me` | 测试 HTTP 代理 |
| `curl -x socks5://user:pass@host:1080 https://ifconfig.me` | 测试 SOCKS5 代理 |
| `proxychains4 curl https://ifconfig.me` | 通过代理链请求 |
| `systemctl restart squid` | 重启 Squid |
| `systemctl restart danted` | 重启 Dante SOCKS5 |
| `journalctl -u squid -f` | 查看 Squid 日志 |
| `journalctl -u danted -f` | 查看 Dante 日志 |
| `tail -f /var/log/squid/access.log` | 查看 Squid 访问日志 |
| `netstat -tulpn \| grep -E '3128|1080|8888'` | 检查代理监听端口 |
| `ss -tulpn \| grep squid` | 检查 Squid 端口 |
| `docker compose -f chain.yml logs -f` | 查看 Docker 代理链日志 |
| `http_proxy=http://host:3128 wget https://example.com` | 环境变量方式使用代理 |
| `export ALL_PROXY=socks5://host:1080` | 设置全局 SOCKS5 代理 |
| `unset http_proxy https_proxy` | 取消代理环境变量 |
| `iptables -t nat -L -n` | 查看 NAT 规则 |
| `iptables -t mangle -L -n` | 查看 mangle 规则 |

---

## 八、高级技巧 | Advanced Tips

### 代理链健康检查 | Proxy Chain Health Check

```bash
#!/bin/bash
# check-chain.sh — Check proxy chain health

CHAIN=("proxy-a:3128" "proxy-b:1080" "proxy-c:3128")
OUTPUT_IP=""  # Should see the exit node's IP

for proxy in "${CHAIN[@]}"; do
    echo -n "Checking $proxy ... "
    if curl -x "http://${proxy}" -o /dev/null -s -w "%{http_code}" \
        --connect-timeout 5 http://httpbin.org/get > /dev/null 2>&1; then
        echo "✅ OK"
    else
        echo "❌ FAILED"
    fi
done

echo "---"
echo "Chain exit IP: $(proxychains4 curl -s https://api.ipify.org)"
```

### 认证与权限 | Authentication and Authorization

**Squid 认证配置:**

```conf
# 使用 NCSA 密码文件
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Proxy - Authenticate
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users
http_access deny all
```

**IP 白名单:**

```conf
acl allowed_ips src 192.168.1.0/24 10.0.0.0/8
http_access allow allowed_ips
http_access deny all
```

### 负载均衡代理链 | Load-balanced Proxy Chain

使用 HAProxy 在前端做负载均衡，后端连接多个代理节点。

```haproxy
# /etc/haproxy/haproxy.cfg
frontend lb-proxy
    bind *:3128
    mode tcp
    default_backend proxy-pool

backend proxy-pool
    mode tcp
    balance roundrobin
    option tcp-check
    server proxy1 proxy-a:3128 check
    server proxy2 proxy-b:3128 check
    server proxy3 proxy-c:3128 check
```

### 数据加密隧道 | Encrypted Tunnels via Proxy

通过 SSH 隧道加密代理链流量：

```bash
# 创建 SSH 隧道到代理链入口
ssh -L 3128:localhost:3128 -N -f user@entry-proxy

# 使用本地端口作为代理
curl -x http://localhost:3128 https://api.ipify.org

# 或通过 SSH 动态转发建立 SOCKS5
ssh -D 1080 -N -f user@entry-proxy
curl -x socks5://localhost:1080 https://api.ipify.org
```

---

## 九、常见问题 | FAQ

### Q1: 代理链连接超时 | Proxy chain connection timeout

**Solution:** 
1. Check each proxy node individually: `curl -x http://node:port -w "%{http_code}" -o /dev/null http://example.com`
2. Increase timeout values in proxychains config
3. Check firewall rules on each proxy node
4. Verify network latency — excessive latency can cause cascading timeouts

### Q2: "SSL certificate problem" 错误 | SSL certificate error

**Solution:**
```bash
# Insecure mode (for testing only)
curl -x http://proxy:3128 -k https://target.com

# Or update CA certificates
apt install -y ca-certificates
update-ca-certificates
```

### Q3: ProxyChains 无法代理 UDP | ProxyChains can't proxy UDP

ProxyChains-NG 不支持 UDP 代理。如果需要 UDP 支持，使用完整 SOCKS5 链或 VPN。

ProxyChains-NG does not support UDP proxying. For UDP support, use a full SOCKS5 chain or VPN.

### Q4: 代理链速度很慢 | Proxy chain is very slow

```bash
# 测试每个节点的延迟
for proxy in ip1 ip2 ip3; do
    time curl -x http://$proxy:3128 -o /dev/null -s http://example.com
done

# 尝试 dynamic_chain 跳过慢节点
# 减小 MTU 以优化延迟
```

### Q5: Squid 报 "FATAL: Unknown cache_peer" 错误

**Solution:** Check that the `cache_peer` directive is properly formatted. The parent proxy must be reachable:

```conf
# Correct format:
cache_peer <hostname_or_ip> parent <port> 0 no-query no-digest proxy-only login=user:pass
```

### Q6: Dante SOCKS5 认证无效 | Dante SOCKS5 authentication fails

```bash
# Check if PAM is properly configured
cat /etc/pam.d/danted

# Or use username-only authentication in danted.conf:
socksmethod: username
# And ensure users exist on the system:
useradd -M -s /sbin/nologin proxyuser
passwd proxyuser
```

### Q7: Docker 中代理无法连接外部 | Proxy in Docker can't reach external

```bash
# Ensure Docker enables IP forwarding on the host
sysctl net.ipv4.ip_forward=1

# If using iptables NAT on host, Docker containers may need extra config
# Add to docker-compose.yml:
# network_mode: "host"
```

### Q8: 代理链中如何调试每级节点 | How to debug each chain node

```bash
# 在每台代理节点上查看实时日志
tail -f /var/log/squid/access.log

# 在出口节点捕获流量
tcpdump -i eth0 -nn 'tcp port 80 or port 443'

# 使用 X-Forwarded-For 头追踪链路
# 客户端可以看到经过的代理 IP 列表
```

---

## ☕ 支持 / Support

如果这个教程对你有帮助，欢迎请我喝杯咖啡：

**USDT (TRC20)**

```
TVbQerV1SF4MXB1JCcAzQxarewHwEPYTKm
```
