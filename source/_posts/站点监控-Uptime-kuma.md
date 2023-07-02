title: 站点监控-Uptime-kuma
author: 顾梦
index_img: 'https://cdn.jishuqin.cn//202306281740421.png'
tags:
  - Uptime-kuma
  - 站点监控
categories:
  - 服务配置
date: 2023-06-28 16:24:36
---
# 站点监控-Uptime-kuma

## 1. 前言

`Uptime-kuma` 是一个基于 Web 的服务器监控工具，用于监视和报告服务器和网络设备的运行时间、性能指标和可用性。它提供了一个直观易用的用户界面，用于实时监控服务器的状态、资源使用情况和响应时间。

`Uptime-Kuma` 的主要特点包括：

- 实时监控：监视服务器和网络设备的运行时间、CPU 和内存使用情况、网络连接、磁盘空间和其他性能指标。
- 报警和通知：设置报警规则，以便在服务器出现问题或达到预定指标时收到通知。可以通过电子邮件、短信或其他集成的通知方式接收警报。
- 历史数据和性能图表：记录服务器的历史性能数据，并生成可视化图表和报告，用于分析和趋势监控。
- 多服务器管理：支持同时监控多台服务器和网络设备，并将它们组织成逻辑分组，方便管理和监控。
- 用户和权限管理：提供用户和权限管理功能，可以根据需要设置不同用户的访问权限和角色。
- 可扩展性：`Uptime-Kuma` 提供了一个开放的 `API` 和插件系统，可以与其他工具和服务进行集成，扩展其功能和定制化需求。

## 2. 部署

<p class="note note-info">本文采用的是 dokcer 容器部署 Uptime-Kuma ，若不想使用docker 来部署可以参考官方文档：<a target="_blank" rel="noopener" href="https://github.com/louislam/uptime-kuma">https://github.com/louislam/uptime-kuma</a>。</p>

### 2.1 拉取镜像

```bash
docker pull louislam/uptime-kuma
```

查看镜像是否拉取到`docker`:

```bash
docker images
```

![](https://cdn.jishuqin.cn//image-20230628164055949.png)

### 2.2 创建映射目录

这里我创建的目录为：`/root/data/kuma/data`，可自行修改，命令如下：

```bash
mkdir -p /root/data/kuma/data
```

### 2.3 运行容器

路径根据你设置的目录修改，端口也可以根据需要修改。

```bash
docker run -d --name kuma --restart=always -p 3001:3001 -v /root/data/kuma/data:/app/data -v /var/run/docker.sock:/var/run/docker.sock  louislam/uptime-kuma
```

## 3.  配置

<p class="note note-warning">运行成功之后，需要开放端口，使 Uptime-Kuma 可以在公网访问。</p>

我这里设置的端口为`3001`,打开 `http://server_ip:3001`访问`Uptime-Kuma`界面。

### 3.1 访问界面

首次访问需设置管理员账号

![](https://cdn.jishuqin.cn//image-20230628165532925.png)

### 3.2 配置反向代理

我使用的是`Nginx Proxy Manager` 配置代理，需在`DNS`服务器添加对应的域名解析记录。 

❗注意：记得开启`Websockets`支持，否则无法正常访问后台。

![](https://cdn.jishuqin.cn//image-20230628170914086.png)

申请`ssl`证书

![](https://cdn.jishuqin.cn//image-20230628170522401.png)

配置完成后，使用域名就可以访问`Kuma`后台了。

### 3.3 添加监控项

登录进后台，点击添加监控项。

![](https://cdn.jishuqin.cn//image-20230628171254914.png)

选择`监控类型`，填入`显示名称` 和 `监控站点路径`。

可根据你实际需求设置其他配置，如：

-  设置心跳检测时间间隔（单位秒）
- 设置证书到期通知
- 设置报警通知，支持多种通知类型

设置完成后就可以看到站点的各项信息了，如下：

![](https://cdn.jishuqin.cn//image-20230628172025154.png)

### 3.4 添加状态页

点击`状态页面` -> `新的状态页`

![](https://cdn.jishuqin.cn//image-20230628172540869.png)

设置状态页的名称和路径

![](https://cdn.jishuqin.cn//image-20230628172650430.png)

添加服务项，其他按需设置即可，点击保存。

![](https://cdn.jishuqin.cn//image-20230628172852407.png)

## 4. 效果

访问对应路径即可查看服务状态信息。

![](https://cdn.jishuqin.cn//image-20230628173052926.png)

效果可以参看我的状态页：https://kuma.jishuqin.cn/status/main

## 参考

[^1]: [教你两分钟快速搭建一个HTTP网站监控程序-Uptime-kuma](https://blog.crazymiao.com/2022/08/uptime-kuma)
[^2]: [云原生之使用docker部署uptime-kuma服务器监控面板](https://bbs.huaweicloud.com/blogs/385898)