# WireGuard 安装脚本使用文档

## 目录

- [简介](#简介)
- [系统要求](#系统要求)
- [快速开始](#快速开始)
- [首次安装](#首次安装)
- [管理菜单](#管理菜单)
- [配置说明](#配置说明)
- [客户端连接](#客户端连接)
- [常见问题](#常见问题)
- [故障排查](#故障排查)
- [卸载](#卸载)

## 简介

这是一个用于在 Linux 服务器上快速部署 [WireGuard](https://www.wireguard.com/) VPN 的 Bash 脚本。

WireGuard 是一种现代点对点 VPN 协议。本脚本部署的模式为：客户端通过加密隧道将所有流量转发到服务器，服务器对客户端流量进行 NAT 转换，使客户端看起来像是使用服务器的 IP 地址访问互联网。

**项目地址**：https://github.com/angristan/wireguard-install

## 系统要求

### 支持的发行版

| 发行版 | 最低版本 |
|--------|---------|
| AlmaLinux | >= 8 |
| Alibaba Cloud Linux | >= 3 |
| Alpine Linux | - |
| Arch Linux | - |
| CentOS Stream | >= 8 |
| Debian | >= 10 |
| Fedora | >= 32 |
| Oracle Linux | - |
| Rocky Linux | >= 8 |
| Ubuntu | >= 18.04 |

### 其他要求

- **root 权限**：脚本必须以 root 用户运行
- **不支持 OpenVZ**：OpenVZ 虚拟化环境不被支持
- **不支持 LXC**：LXC 容器环境暂不支持（需要在宿主机安装内核模块）
- **公网 IP**：服务器需要拥有公网 IPv4 或 IPv6 地址

## 快速开始

```bash
# 下载脚本
curl -O https://raw.githubusercontent.com/yangxinbo/wireguard-install/master/wireguard-install.sh

# 赋予执行权限
chmod +x wireguard-install.sh

# 运行脚本
./wireguard-install.sh
```

## 首次安装

首次运行脚本时，会引导你完成以下配置：

### 1. 服务器公网地址

脚本会自动检测服务器的公网 IPv4 地址，你也可以手动输入 IPv6 地址。

```
IPv4 or IPv6 public address: [自动检测的地址]
```

### 2. 公网网卡接口

脚本会自动检测默认路由的网卡名称（如 `eth0`、`ens3` 等）。

```
Public interface: [自动检测的网卡]
```

### 3. WireGuard 接口名称

自定义 WireGuard 虚拟网卡名称，默认为 `wg0`，不超过 15 个字符。

```
WireGuard interface name: wg0
```

### 4. 服务器 WireGuard IP

设置服务器在 WireGuard 网络中的 IP 地址：

- **IPv4**：默认 `10.66.66.1`（/24 网段，支持 253 个客户端）
- **IPv6**：默认 `fd42:42:42::1`（/64 网段）

### 5. 监听端口

设置 WireGuard 监听的 UDP 端口，范围 1-65535，默认随机生成一个私有端口（49152-65535）。

### 6. DNS 解析器

设置客户端通过 VPN 使用的 DNS 服务器：

- 默认主 DNS：`1.1.1.1`（Cloudflare）
- 默认备 DNS：`1.0.0.1`（Cloudflare）

可改为其他 DNS，如：
- `8.8.8.8` / `8.8.4.4`（Google）
- `223.5.5.5` / `223.6.6.6`（阿里云）
- `119.29.29.29`（腾讯云）

### 7. AllowedIPs

WireGuard 的 `AllowedIPs` 参数决定哪些流量走 VPN：

- **默认** `0.0.0.0/0,::/0`：所有流量走 VPN（全量路由）
- **自定义网段**：仅指定网段走 VPN（分流模式）

### 安装完成后

脚本会自动：
1. 安装 WireGuard 内核模块和工具
2. 生成服务器密钥对
3. 创建 `/etc/wireguard/wg0.conf` 配置文件
4. 配置防火墙规则（iptables 或 firewalld）
5. 启用 IP 转发
6. 启动 WireGuard 服务并设置开机自启
7. 创建第一个客户端

## 管理菜单

再次运行脚本时，会显示管理菜单：

```
Welcome to WireGuard-install!
It looks like WireGuard is already installed.
What do you want to do?
   1) Add a new user
   2) List all users
   3) Revoke existing user
   4) Uninstall WireGuard
   5) Exit
```

### 1. 添加新用户

为新的客户端生成配置文件：

1. 输入客户端名称（字母、数字、下划线、连字符，不超过 15 字符）
2. 选择客户端 IPv4 地址（自动分配，也可手动指定）
3. 选择客户端 IPv6 地址（与 IPv4 后缀一致）
4. 生成客户端配置文件和二维码

配置文件保存路径：
- 如果客户端名对应用户名：`/home/<用户名>/wg0-client-<名称>.conf`
- 否则：`/root/wg0-client-<名称>.conf` 或 `/home/<SUDO_USER>/wg0-client-<名称>.conf`

### 2. 列出所有用户

显示当前所有已配置的客户端列表。

### 3. 撤销用户

移除指定客户端：

1. 选择要撤销的客户端编号
2. 脚本会从服务器配置中移除该客户端的 Peer 配置
3. 删除对应的客户端配置文件
4. 重新加载 WireGuard 配置（无需重启服务）

### 4. 卸载 WireGuard

⚠️ **警告**：此操作会删除所有 WireGuard 配置文件，请先备份 `/etc/wireguard` 目录。

### 5. 退出

退出脚本。

## 配置说明

### 配置文件位置

| 文件 | 说明 |
|------|------|
| `/etc/wireguard/wg0.conf` | 服务器端配置 |
| `/etc/wireguard/params` | 安装参数（脚本内部使用） |
| `/etc/sysctl.d/wg.conf` | IP 转发配置 |

### 服务器配置结构

```ini
[Interface]
Address = 10.66.66.1/24,fd42:42:42::1/64
ListenPort = 51820
PrivateKey = <服务器私钥>
PostUp = <防火墙规则>
PostDown = <防火墙清理规则>

### Client client1
[Peer]
PublicKey = <客户端公钥>
PresharedKey = <预共享密钥>
AllowedIPs = 10.66.66.2/32,fd42:42:42::2/128
```

### 客户端配置结构

```ini
[Interface]
PrivateKey = <客户端私钥>
Address = 10.66.66.2/32,fd42:42:42::2/128
DNS = 1.1.1.1,1.0.0.1

[Peer]
PublicKey = <服务器公钥>
PresharedKey = <预共享密钥>
Endpoint = 1.2.3.4:51820
AllowedIPs = 0.0.0.0/0,::/0
```

## 客户端连接

### 下载配置文件

将生成的 `wg0-client-<名称>.conf` 文件传输到客户端设备。

```bash
# 在服务器上查看文件位置
cat /root/wg0-client-<名称>.conf

# 或使用 scp 下载
scp root@server:/root/wg0-client-<名称>.conf ./
```

### 各平台连接方式

#### Linux

```bash
# 安装 WireGuard
sudo apt install wireguard      # Debian/Ubuntu
sudo dnf install wireguard      # Fedora
sudo pacman -S wireguard        # Arch

# 复制配置文件
sudo cp wg0-client.conf /etc/wireguard/

# 启动连接
sudo wg-quick up wg0-client

# 断开连接
sudo wg-quick down wg0-client
```

#### Windows

1. 下载 [WireGuard for Windows](https://www.wireguard.com/install/)
2. 打开 WireGuard 客户端
3. 导入 `wg0-client-<名称>.conf` 文件
4. 激活连接

#### macOS

1. 通过 Homebrew 安装：`brew install wireguard-tools`
2. 或使用 [WireGuard for macOS](https://www.wireguard.com/install/)
3. 导入配置文件并激活

#### Android / iOS

1. 安装 WireGuard App
2. 扫描安装脚本输出的二维码，或导入配置文件
3. 激活连接

## 常见问题

### Q: 脚本提示 "You need to run this script as root"

确保使用 root 用户运行，或以 sudo 执行：

```bash
sudo ./wireguard-install.sh
```

### Q: 客户端连接后无法上网

1. 检查服务器防火墙是否放行 WireGuard 端口：
   ```bash
   # iptables
   sudo iptables -L INPUT -n | grep <端口号>
   # firewalld
   sudo firewall-cmd --list-ports
   ```

2. 检查云服务商的安全组规则，确保 UDP 端口已放行

3. 检查 IP 转发是否启用：
   ```bash
   sysctl net.ipv4.ip_forward
   sysctl net.ipv6.conf.all.forwarding
   # 应返回 1
   ```

4. 尝试重启服务器

### Q: 提示 "Cannot find device wg0"

内核更新后可能需要重启服务器以加载新的 WireGuard 模块：

```bash
reboot
```

### Q: 如何查看客户端连接状态

```bash
wg show
# 或
wg
```

输出示例：
```
interface: wg0
  public key: xxxxx
  listening port: 51820

peer: xxxxx
  endpoint: 203.0.113.1:54321
  allowed ips: 10.66.66.2/32, fd42:42:42::2/128
  transfer: 1.2 GiB received, 3.4 GiB sent
  latest handshake: 5 minutes ago
```

### Q: 最大支持多少个客户端

默认 /24 网段支持 **253** 个客户端（10.66.66.2 ~ 10.66.66.254）。

### Q: 如何修改 MTU

在客户端配置文件中取消注释 MTU 行：

```ini
[Interface]
# ...
MTU = 1420
```

可使用 [wg-mtu-finder](https://github.com/nitred/nr-wg-mtu-finder) 工具找到最佳 MTU 值。

### Q: 如何配置分流（只让特定流量走 VPN）

修改客户端配置文件中的 `AllowedIPs`：

```ini
[Peer]
# 只让访问内网网段的流量走 VPN
AllowedIPs = 10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
```

## 故障排查

### 查看服务状态

```bash
# systemd 系统
systemctl status wg-quick@wg0

# Alpine Linux
rc-service wg-quick.wg0 status
```

### 查看日志

```bash
journalctl -u wg-quick@wg0 -f
```

### 测试 IP 转发

```bash
# 检查 IPv4 转发
cat /proc/sys/net/ipv4/ip_forward

# 检查 IPv6 转发
cat /proc/sys/net/ipv6/conf/all/forwarding
```

### 检查防火墙规则

```bash
# iptables
iptables -L -n -v
iptables -t nat -L -n -v

# firewalld
firewall-cmd --list-all
```

### 常用管理命令

```bash
# 启动 WireGuard
sudo systemctl start wg-quick@wg0

# 停止 WireGuard
sudo systemctl stop wg-quick@wg0

# 重启 WireGuard
sudo systemctl restart wg-quick@wg0

# 查看运行状态
sudo systemctl status wg-quick@wg0

# 查看连接信息
sudo wg show

# 手动重载配置
sudo wg syncconf wg0 <(sudo wg-quick strip wg0)
```

## 卸载

运行脚本选择选项 4，或手动卸载：

```bash
# 1. 停止服务
systemctl stop wg-quick@wg0
systemctl disable wg-quick@wg0

# 2. 删除配置文件
rm -rf /etc/wireguard
rm -f /etc/sysctl.d/wg.conf

# 3. 重载 sysctl
sysctl --system

# 4. 卸载软件包（以 Ubuntu 为例）
apt-get remove -y wireguard wireguard-tools qrencode
```

## 参考链接

- [WireGuard 官网](https://www.wireguard.com/)
- [WireGuard 文档](https://www.wireguard.com/docs/)
- [项目 GitHub 仓库](https://github.com/angristan/wireguard-install)
- [OpenVPN 安装脚本](https://github.com/angristan/openvpn-install)