---
title: "Linux常用命令"
date: 2025-08-04T14:55:29+08:00
draft: false
tags: [linux]
---

## Linux根目录

|    文件夹名    |                   解释                   |
| :------------: | :--------------------------------------: |
| bin或者usr/bin |   存放命令，相当于Windows下的环境变量    |
|      usr       |           using soft resource            |
|      var       |   系统文件，系统运行过程相关(variable)   |
|      etc       |                 配置文件                 |
|      root      |                管理员目录                |
|      home      |               普通用户目录               |
|      opt       |              第三方软件目录              |
|     media      |                可删除设备                |
|      mnt       |                 早期命令                 |
| lib或者usr/lib |     存放linux系统函数有64和32的区分      |
|      proc      |         与虚拟文件系统和内核有关         |
|      boot      |                 字面意思                 |
|      dev       | 在Linux里所有设备，跟Windows下的驱动类似 |
|      sys       |               系统常用文件               |
|      tmp       |                 临时文件                 |

## 常用命令

### 防火墙

`systemctl disable firewalld` 关闭防火墙，需要重启生效 `reboot`

`systemctl start firewalld` 开启防火墙

