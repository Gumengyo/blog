title: 使用Cloudreve搭建自己的网盘
author: 顾梦
tags:
  - Cloudreve
index_img: 'https://cdn.jishuqin.cn/cloudreve.png'
categories:
  - 服务配置
date: 2023-03-02 11:21:00
---

# 一、前言

Cloudreve 可以让您快速搭建起公私兼备的网盘系统。Cloudreve 在底层支持不同的云存储平台，用户在实际使用时无须关心物理存储方式。你可以使用 Cloudreve 搭建个人用网盘、文件分享系统，亦或是针对大小团体的公有云系统。

![](https://cdn.jishuqin.cn/2023-03-03_21-10-31.png)

## 二、安装Cloudreve

```
# 新建一个文件夹存放程序
mkdir /www/wwwroot/cloudreve 

#进入该文件夹
cd /www/wwwroot/cloudreve

# 下载对应资源包（选择对应系统版本的资源包）
wget https://github.com/cloudreve/Cloudreve/releases/download/3.7.1/cloudreve_3.7.1_linux_amd64.tar.gz

# 解压获取到的主程序
tar -zxvf cloudreve_3.7.1_linux_amd64.tar.gz

# 赋予执行权限
chmod +x ./cloudreve

# 启动 Cloudreve
./cloudreve
```

<p class="note note-warning">123</p>

