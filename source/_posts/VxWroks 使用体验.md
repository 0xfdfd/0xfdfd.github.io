---
title: "VxWroks 使用体验"
date: 2023-01-31T14:57:25+08:00
tags:
---

## 关于

主页：[Wind River VxWorks](http://www.windriver.com/products/vxworks/)

> 所有的使用体验基于 VxWorks Version 22.09。

<!-- more -->

## 运行平台

VxWorks 官方提供的非商业 SDK 可以基于如下平台运行：

+ QEMU (x86-64)
+ QEMU (sabrelite)
+ [Raspberry Pi 4B][1]
+ [UP Squared](https://up-board.org/upsquared/specifications/)
+ [NXP i.MX 8M Quad Evaluation Kit (EVK)](https://www.nxp.com/design/development-boards/i-mx-evaluation-and-development-boards/evaluation-kit-for-the-i-mx-8m-applications-processor:MCIMX8M-EVK)
+ [Microchip Polarfire SoC Icicle Kit (RISC-V)](https://www.microchip.com/en-us/development-tool/MPFS-ICICLE-KIT-ES)
+ [Sifive Hifive Unleashed (RISC-V)](https://www.sifive.com/boards/hifive-unleashed)
+ [Sifive Hifive Unmatched (RISC-V)](https://www.sifive.com/boards/hifive-unmatched)

从官方支持的平台来看，对应硬件内存都十分的庞大，最小的也是 [Raspberry Pi 4B][1] 的 1GB 内存，这可能与其支持的特性有关，比如：
+ [OCI 容器](https://opencontainers.org/)
+ C++ 17
+ Boost
+ Rust
+ Python 3.9
+ pandas

VxWorks 提供的 SDK 已经将上述环境（包括编译工具链）全部打包，因此比较庞大。

## 开发环境搭建

Wind River 提供 [Wind River Studio](https://www.windriver.com/studio) 用以在线开发。依照官方文档，其提供了开发、测试、部署、运行、监控、维护的完整链路服务。由于没有找到试用入口，这里我只能利用 SDK 搭建本地运行开发环境。SDK 下载地址：[vxworks-software-development-kit-sdk](https://forums.windriver.com/t/vxworks-software-development-kit-sdk/43)。

VxWorks 的开发环境需要搭建在 Linux 下。SDK 的使用仅需引入对应环境变量即可：
```
$ source sdkenv.sh
```

<!-- URL -->
[1]: https://www.raspberrypi.com/products/raspberry-pi-4-model-b/

