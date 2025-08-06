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

`systemctl status firewalld` 查看防火墙状态

`  firewall-cmd --list-port` 查询已经开放的端口

### 解压缩

zip格式：

+ `zip -r  压缩包名 .zip  文件路径`
+ `unzip 压缩包名 .zip`

tar格式(更常用):

+ 打成tar包：`tar -cvf  压缩包名 .tar  文件路径`
+ 解包：`  tar -xvf  包名 .tar  解包后的名`
+ 打成tar压缩包`  tar -zcvf  包名 .tar 文件名`
+ 解压：`tar -xzvf  包名 .tar  解包后的名`

### 其他

发送文件：scp  哪个⽂件 / 压缩包  对⽅⽤户名 @ 对⽅主机 : 发到哪个位置

查看磁盘文件：df 

输入过的命令：history

查看ip：ip addr

文件详细信息：ll == ls -l

查看文件：

+ tail 文件名   查看文件最后10行 tail -n 文件名    查看文件后n行
+ cat 全部展示
+ less 显示一页

查看进程：ps aux 可以配合grep 使用
