# WireGuard 安装脚本

![Lint](https://github.com/angristan/wireguard-install/workflows/Lint/badge.svg)
[![](https://img.shields.io/badge/Say%20Thanks-!-1EAEDB.svg)](https://saythanks.io/to/angristan)

**本项目是一个 Bash 脚本，旨在以最简单的方式在 Linux 服务器上部署 [WireGuard](https://www.wireguard.com/) VPN！**

WireGuard 是一种点对点 VPN。这里所说的 VPN 是指：客户端将通过加密隧道将所有流量转发到服务器。
服务器会对客户端的流量进行 NAT 转换，使其看起来像是客户端使用服务器的 IP 地址浏览互联网。

该脚本同时支持 IPv4 和 IPv6。请查看 [issues](https://github.com/angristan/wireguard-install/issues) 了解正在进行的开发、Bug 和计划功能！你也可以在 [discussions](https://github.com/angristan/wireguard-install/discussions) 中寻求帮助。

WireGuard 不适合你的使用场景？试试 [openvpn-install](https://github.com/angristan/openvpn-install)。

## 系统要求

支持的发行版：

- AlmaLinux >= 8
- Alibaba Cloud Linux >= 3
- Alpine Linux
- Arch Linux
- CentOS Stream >= 8
- Debian >= 10
- Fedora >= 32
- Oracle Linux
- Rocky Linux >= 8
- Ubuntu >= 18.04

## 使用方法

下载并执行脚本。回答脚本提出的问题，它将自动完成剩余配置。

```bash
curl -O https://raw.githubusercontent.com/yangxinbo/wireguard-install/master/wireguard-install.sh
chmod +x wireguard-install.sh
./wireguard-install.sh
```

它将在服务器上安装 WireGuard（内核模块和工具），完成配置，创建 systemd 服务以及客户端配置文件。

再次运行脚本即可添加或删除客户端！

## 推荐服务商

推荐以下性价比高的云服务器提供商用于搭建 VPN 服务器：

- [Vultr](https://www.vultr.com/?ref=8948982-8H)：全球节点，支持 IPv6，$5/月起
- [Hetzner](https://hetzner.cloud/?ref=ywtlvZsjgeDq)：德国、芬兰和美国节点，支持 IPv6，20 TB 流量，4.5€/月起
- [Digital Ocean](https://m.do.co/c/ed0ba143fe53)：全球节点，支持 IPv6，$4/月起

## 贡献指南

欢迎贡献！你可以通过以下方式参与：

### 讨论变更

在提交 PR 之前，请先创建一个 issue 讨论你的变更方案，尤其是较大的变更。

### 代码格式

我们使用 [shellcheck](https://github.com/koalaman/shellcheck) 和 [shfmt](https://github.com/mvdan/sh) 来强制执行 Bash 代码风格规范。每次提交 / PR 都会通过 GitHub Actions 执行检查，配置见 [这里](https://github.com/angristan/wireguard-install/blob/master/.github/workflows/lint.yml)。

## 致谢

如果你认可这个项目，可以[点这里](https://saythanks.io/to/angristan)说声谢谢！

## 许可

本项目采用 [MIT 许可证](https://raw.githubusercontent.com/angristan/wireguard-install/master/LICENSE)

## Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=angristan/wireguard-install&type=Date)](https://star-history.com/#angristan/wireguard-install&Date)