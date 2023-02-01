---
title: "VMware 配置静态 IP"
date: 2023-02-01T09:17:34+08:00
tags:
---

## 背景

默认情况下，VMware 中的虚拟机使用 NAT 方式连入网络，其 IP 分配方式是 `DHCP` 。为了方便虚拟机之间相互通信，想要改成使用静态 IP 分配的方式。

<!-- more -->

## VMware 中的 IP 区段划分

> 来源：[how to give guest a static ip address](https://communities.vmware.com/t5/VMware-Workstation-Pro/how-to-give-guest-a-static-ip-address/td-p/803481)

Host-Only Network:
| Range                     | Address Use      | Example                       |
| ------------------------- | ---------------- | ----------------------------- |
| `<net>`.1                 | Host Machine     | 192.168.0.1                   |
| `<net>`.2 - `<net>`.127   | Static Addresses | 192.168.0.2 - 192.168.0.127   |
| `<net>`.128 - `<net>`.253 | DHCP Assigned    | 192.168.0.128 - 192.168.0.253 |
| `<net>`.254               | DHCP Server      | 192.168.0.254                 |
| `<net>`.255               | Broadcasting     | 192.168.0.255                 |

NAT Network:
| Range                     | Address Use      | Example                       |
| ------------------------- | ---------------- | ----------------------------- |
| `<net>`.1                 | Host Machine     | 192.168.0.1                   |
| `<net>`.2                 | NAT Device       | 192.168.0.2                   |
| `<net>`.3 - `<net>`.127   | Static Addresses | 192.168.0.3 - 192.168.0.127   |
| `<net>`.128 - `<net>`.253 | DHCP Assigned    | 192.168.0.128 - 192.168.0.253 |
| `<net>`.254               | DHCP Server      | 192.168.0.254                 |
| `<net>`.255               | Broadcasting     | 192.168.0.255                 |

## 静态地址分配

### 方法1：修改客户机配置

> 来源：[Network configuration](https://ubuntu.com/server/docs/network-configuration)

通过上面的 IP 区段划分表格，我们需要关注两个事项：
1. NAT 网络中，网关地址不再是 `<net>.1`，而是 `<net>.2`。`<net>.1` 指代的是宿主机 IP 地址。
2. NAT 网络默认已经预留 `<net>.3 - <net>.127` 作为静态 IP 地址段

因此我们在客户机中做出网络配置如下：

```
$ cat /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: false
      addresses:
        - 192.168.108.10/24
      routes:
        - to: default
          via: 192.168.108.2
      nameservers:
        addresses: [192.168.108.2]
```

### 方法2：修改 VMware 配置

> 来源：[Assign static IP to guest with NAT Virt Network Adaptor?](https://communities.vmware.com/t5/VMware-Workstation-Player/Assign-static-IP-to-guest-with-NAT-Virt-Network-Adaptor/td-p/2583621)

修改 `vmnetdhcp.conf` 文件（位于 VMware 安装目录下），在 `#END` 前添加如下内容：

```
host LuckyLuke {
    hardware ethernet 00:0c:29:23:b6:12;
    fixed-address 192.168.156.77;
}
```

`LuckyLuke` 可以是任意名字，只要保证文件内唯一即可。`hardware ethernet` 为客户机 MAC 地址，`fixed-address` 为分配的静态 IP 地址。
