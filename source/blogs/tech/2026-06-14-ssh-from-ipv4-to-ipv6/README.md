# 仅 IPv4 的家用电脑访问纯 IPv6 的公网服务器方法
现有一台私有服务器，只有纯 ipv6，我需要通过有 ipv6 的机器访问即可，但现在问题是 ipv6 并不是所有设备都有。工位电脑只有 ipv4，无法访问服务器。所以我为什么不给我的服务器加一个 ipv4？因为服务器是免费的，加装 ipv4 需要每个月多支付几美元。

解决方法，使用一台带有双栈的机器做数据中转就可以了，但是我既然能买双栈，就不会白嫖服务器了，说到底还是一个字，穷。😅

中转机器还真有免费的，就是 cloudflare，但是需要一个额外的域名。这个也可以接受，域名比机器便宜多了，有些年付可以低至一元。

> 需要注意，cloudflare 只能中转http 和 https 流量。。并且只能中转某些发往固定端口的流量。


## 1. 问题场景

- **家用电脑**：只有 IPv4 地址（可能位于 NAT 之后），无法直接发起 IPv6 连接。
- **公网服务器**：拥有公网 IPv6 地址，但**没有**公网 IPv4 地址（或 IPv4 被防火墙屏蔽）。
- **需求**：希望通过 SSH 从 IPv4 电脑安全地管理纯 IPv6 服务器。

额外限制可能包括：
- 服务器所在网络仅允许少数端口（如 80、443 等）对外暴露。
- 需要加密通信，避免内容被中间网络截获。

---

## 2. 解决方案概述

利用 **Cloudflare** 的免费代理服务，将 IPv6 服务器映射到一个 IPv4 可达的域名，再通过 **WebSocket over TLS（WSS）** 将 SSH 流量封装成 HTTPS 流量，从而使得 SSH 流量穿透 Cloudflare。

**数据流向**：
```
[本地 IPv4 电脑] 
  → SSH 客户端连接本地 2253 端口
  → websocat 将 SSH 流量转成 WebSocket  # 为何要转换成 ws 流量，是因为 Cloueflare 免费版不支持 SSH 
  → wss://托管到Cloueflare的域名:2053
  → IPv6 服务器的 2053 端口
  → websockify 从 2053 端口接收到数据后，解包并转发到 [::1]:22
  → SSH 服务响应
```

**核心组件**：
- **Cloudflare DNS 代理**：将域名解析到服务器 IPv6 地址，并代理 HTTPS/WSS 流量。
- **websockify**（服务端）：监听 Cloudflare 支持的 HTTPS 端口，将 WebSocket 连接转换为 TCP 连接。
- **websocat**（客户端）：在 Windows 上将本地 TCP 流量通过 WebSocket 隧道发送。
- **SSH 客户端**：连接本地转发端口。

---

## 3. Cloudflare 端配置（先决条件）

### 3.1 添加 DNS 记录并启用代理

1. 登录 Cloudflare 控制台，选择你的域名（例如 `example.com`）。
2. 进入 **DNS** → **Records**，添加一条 **AAAA** 记录：
   - **名称**：例如 `test`（完整域名 `xxx.xxx.com`）
   - **IPv6 地址**：你的服务器公网 IPv6 地址（如 `2001:db8::1`）
   - **代理状态**：必须开启 **橙色云朵（Proxied）**，这样流量才会经过 Cloudflare。
3. 等待 DNS 解析生效（通常 1~5 分钟）。

![[Cloudflare-proxyYourDomain.png]]

> 此步骤可以参考其它博客进行配置，显示橙色云朵即可。

### 3.2 生成 Origin 证书（用于服务端 TLS）

为了让 Cloudflare 到源站的连接也加密（Full (strict) 模式要求），需要生成并安装 Origin Certificate。

1. 在 Cloudflare 控制台进入 **SSL/TLS** → **Origin Server**。
2. 点击 **Create Certificate**，选择“Let Cloudflare generate a private key and CSR”。
3. 有效期建议保留默认（15 年），点击 **Next**。
4. 下载两个文件：
   - **Origin Certificate**（`cert.pem`）
   - **Private Key**（`key.pem`）

![[Cloudflare-genOriginServerCert.png]]

   > 注意：私钥仅在此时可见，关闭窗口后私钥无法再次复制，请妥善保存。我这里添加了两个域名，通常一个即可，或者使用通配符 `*.xxx.com`。

### 3.3 设置 SSL/TLS 加密模式

进入 **SSL/TLS** → **Overview**，将加密模式设置为 **Full (strict)**。  
这要求源站必须提供有效的 TLS 证书（即你刚刚生成并安装的 Origin Certificate），且 Cloudflare 会严格验证。

![[Cloudflare-setSslTlsEncrypt.png]]

### 3.4 确认端口支持

Cloudflare 代理仅允许特定端口，建议使用 **HTTPS 端口** 中的 `2053`、`2083`、`2087`、`2096` 或 `8443`。  
本例选用 **2053**。

---

## 4. 服务端配置（纯 IPv6 服务器）

以下操作在服务器（Ubuntu 22.04）上以 `root` 或 `sudo` 执行。

### 4.1 安装 websockify

```bash
# 使用系统包管理器（推荐）
apt update && apt install websockify -y
```

![[Websockify-usage.png]]

### 4.2 放置 TLS 证书

将上一步从 Cloudflare 下载的 `cert.pem` 和 `key.pem` 上传到服务器，并设置正确权限：

```bash
mkdir -p /etc/websockify/ssl
chmod 700 /etc/websockify/ssl

# 将证书内容分别保存到文件（使用 vi 或 scp）
vi /etc/websockify/ssl/cert.pem
vi /etc/websockify/ssl/key.pem

# 创建专门运行服务的用户
useradd -r -s /bin/false websockify

# 调整权限：私钥仅允许 websockify 组读取
chown root:websockify /etc/websockify/ssl/cert.pem /etc/websockify/ssl/key.pem
chmod 644 /etc/websockify/ssl/cert.pem
chmod 640 /etc/websockify/ssl/key.pem
```

![[Websockify-setSslCertPermission.png]]

### 4.3 创建 systemd 服务

编辑 `/etc/systemd/system/websockify.service`：

![[Websockify-createLinuxSystemService.png]]

- `[::]:2053`：监听所有 IPv6 接口。
- `[::1]:22`：转发到本地 SSH 端口（确保 SSH 服务在监听）。
- `--ssl-only`：强制 TLS，不接受明文连接。
- `--log-file`：指定日志文件。

创建日志目录并授权：

```bash
mkdir -p /var/log/websockify
chown websockify:websockify /var/log/websockify
```

![[Websockify-configureLogDirectoryPermission.png]]

重载、启用并启动：

```bash
systemctl daemon-reload
systemctl enable websockify
systemctl start websockify
systemctl status websockify
```

![[Websockify-showServiceStatus.png]]

### 4.4 防火墙及安全加固（推荐）

仅允许 Cloudflare 的 IP 段访问 2053 端口，避免源站暴露。

```bash
# 下载 Cloudflare IP 列表
curl -s https://www.cloudflare.com/ips-v4 -o /tmp/cf_ips_v4
curl -s https://www.cloudflare.com/ips-v6 -o /tmp/cf_ips_v6

# 允许 Cloudflare IP 访问 2053（使用 iptables/ip6tables）
for ip in $(cat /tmp/cf_ips_v4); do
    iptables -A INPUT -p tcp -s $ip --dport 2053 -j ACCEPT
done
for ip in $(cat /tmp/cf_ips_v6); do
    ip6tables -A INPUT -p tcp -s $ip --dport 2053 -j ACCEPT
done
# 拒绝其他来源
iptables -A INPUT -p tcp --dport 2053 -j DROP
ip6tables -A INPUT -p tcp --dport 2053 -j DROP
```

> 如果使用 `ufw`，可用脚本循环添加 allow from 规则。

---

## 5. 客户端配置（Windows 仅 IPv4 电脑）

### 5.1 下载 websocat

从 [https://github.com/vi/websocat/releases](https://github.com/vi/websocat/releases) 下载 `websocat.exe`，放到 `C:\Users\你的用户名\` 或加入 PATH。

### 5.2 启动本地隧道

打开命令提示符（CMD），执行：

```cmd
websocat --binary tcp-listen:127.0.0.1:2253 wss://xxx.xxx.site:2053
```

- `--binary`：确保二进制数据正确传输。
- `tcp-listen:127.0.0.1:2253`：在本地监听 `2253` 端口，监听本地的哪个端口随便选。
- `wss://xxx.xxx.site:2053`：域名被托管后，访问域名的 2053 端口，流量会被转发到服务器的 2053端口。

![[Websocat-listeningPort.png]]

保持该窗口运行。

### 5.3 连接 SSH

打开另一个 CMD 窗口，使用 SSH 客户端（如 OpenSSH）：

```cmd
ssh -p 2253 用户名@127.0.0.1
```

输入密码或密钥后即可登录服务器。

![[SSH-SuccessfulConnection.png]]

---

## 6. 调试与详细日志（可选）

若连接失败，可在服务端启用更详细日志：

1. 修改 systemd 服务，在 `ExecStart` 中添加 `--verbose`（或 `-vv`）选项。
2. 还可添加 `--record /var/log/websockify/sessions/ws-record` 记录原始 WebSocket 帧。
3. 重启服务，并查看日志：
   ```bash
   journalctl -u websockify -f
   tail -f /var/log/websockify/websockify.log
   ```

客户端可通过 `websocat` 的 `-v` 参数增加输出。

---

## 7. 常见问题排查

| 错误现象 | 可能原因 | 解决方法 |
|---------|---------|----------|
| Cloudflare 返回 **525** | SSL 握手失败 | 检查证书与私钥是否匹配；权限是否允许 websockify 用户读取；SSL/TLS 模式是否为 Full (strict)。 |
| Cloudflare 返回 **521** | 源站不可达 | 确认防火墙放行 2053；websockify 是否正确监听；安全组规则允许 Cloudflare IP。 |
| 服务日志报 `Permission denied` | 目标地址 `::1:22` 连接被拒 | 改用 `127.0.0.1:22`，并确保 SSH 监听 IPv4 回环。 |
| `websocat` 连接超时 | 域名解析到 Cloudflare 但端口不通 | 确认 Cloudflare 支持该端口；检查代理状态是否为橙色云朵。 |
| SSH 连接后立即断开 | WebSocket 隧道未建立 | 检查 `websocat` 是否使用 `wss://`（SSL），服务端是否开启 `--ssl-only`。 |

---

## 8. 参考资料

- [Cloudflare 支持的端口列表](https://developers.cloudflare.com/fundamentals/reference/network-ports/)
- [websockify 官方文档](https://github.com/novnc/websockify)
- [websocat 项目](https://github.com/vi/websocat)
