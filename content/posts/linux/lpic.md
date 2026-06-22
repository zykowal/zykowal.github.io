---
date: 2026-06-22
draft: false
title: 'LPIC'
tags: ["Cheatsheet"]
categories: ["Linux"]
series: []
summary: "LPIC"
---

## LPIC

### 什么是 Linux Distribution

Linux Distribution（发行版）是由 Linux 内核、软件包管理器、桌面环境、系统工具和应用程序组成的一个完整操作系统。常见的发行版有 Debian（及其衍生版 Ubuntu）、Red Hat（及其衍生版 Fedora、CentOS/RHEL）、Arch Linux、openSUSE 等。不同的发行版面向不同的用户群体，例如 Ubuntu 注重易用性，Arch 追求简洁与高度可定制。

### Basic Command

#### man

`man` 是 manual 的缩写，用于查看 Linux 命令、系统调用、库函数等的帮助手册。手册按章节分类（如 1 用户命令、2 系统调用、3 C 库函数），可通过 `man ls`、`man 3 printf` 等命令查阅。使用 `man -k keyword` 可以搜索相关手册页。

#### lspci

列出系统中所有 PCI 总线上的设备信息，包括显卡、网卡、声卡等。常用于硬件识别与故障排查，配合 `-v`（详细）、`-vv`（更详细）可查看设备驱动和资源分配情况。

#### lsusb

列出所有连接到 USB 总线上的设备。通过 `-v` 查看详细描述符信息，`-t` 以树形结构显示 USB 设备拓扑，常用于调试外设驱动或确认 USB 设备是否被正确识别。

#### lsblk

列出所有块设备（如硬盘、分区、SSD、USB 存储等），默认以树形结构显示设备间的依赖关系（如 `sda` 下的 `sda1`、`sda2`）。可通过 `-f` 显示文件系统类型和 UUID，`-m` 显示权限和所有者信息。

### Virtual Directory

#### /proc

一个虚拟文件系统，**不占用磁盘空间**，由内核动态生成，用于暴露内核和进程的运行时信息。例如 `/proc/cpuinfo`（CPU 信息）、`/proc/meminfo`（内存信息）、`/proc/[pid]/`（进程详细信息）。也可通过写入某些文件来修改内核参数（如 `/proc/sys/`）。

#### /sys

也是一个虚拟文件系统，相比 `/proc` 更加结构化，以设备模型（kobject）为核心导出内核对象信息，用于管理和查看硬件设备、驱动、总线等信息。常用于 power management、device driver 开发，以及 udev 规则编写。

#### /dev

存放设备文件的目录，每个文件对应系统中的一个设备。传统上使用主设备号和次设备号标识设备类型。主要分为：

- **字符设备**（如 `/dev/tty`、`/dev/random`）
- **块设备**（如 `/dev/sda`）
- 现代 Linux 通过 `udev` 动态管理 `/dev` 下的设备节点。

### Kernel Module

内核模块（Kernel Module）是可以在不重新编译内核的情况下动态加载和卸载的代码片段，通常用于添加硬件驱动、文件系统支持等。它们被编译为 `.ko`（kernel object）文件，存放在 `/lib/modules/$(uname -r)/` 目录下。

#### modprobe

用于加载或卸载内核模块及其依赖，比 `insmod`/`rmmod` 更智能——它会自动处理模块间的依赖关系。配置文件位于 `/etc/modprobe.d/` 和 `/lib/modprobe.d/`，可用于设置模块参数或黑名单。

#### lsmod

列出当前已加载的内核模块，以列表形式显示模块名称、大小、被其他模块引用的次数以及依赖模块列表。实际上通过读取 `/proc/modules` 获取信息。

#### modinfo

显示内核模块的详细信息，包括作者、描述、许可证、依赖、参数和别名等。用法如 `modinfo <module_name>`，可配合 `-p` 仅查看模块参数。

#### uname

打印系统信息，常用选项：

- `-r`：内核版本号
- `-a`：全部信息（内核名称、主机名、版本、硬件架构等）
- `-m`：硬件架构（如 x86_64、aarch64）
- 常用于确认当前系统环境，决定下载哪个版本的驱动或软件包。
