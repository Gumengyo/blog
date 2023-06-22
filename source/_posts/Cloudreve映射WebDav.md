title: Cloudreve映射WebDav
author: 顾梦
tags:
  - Cloudreve
index_img: 'https://cdn.jishuqin.cn/cloudreve.png'
categories:
  - 服务配置
date: 2023-03-04 09:57:00
---
# 一、前言

WebDAV 是一种基于 HTTP 协议的文件传输协议，如今有许多第三方文件管理器、视频播放器等产品都支持通过 WebDAV 协议访问 Cloudreve 中的文件，你可以借此实现跨平台的文件共享与同步。

# 二、添加权限

登录网盘进入管理面板，点击左侧用户组，点击编辑按钮为对应用户组开启WebDAV协议连接网盘功能。

![](https://cdn.jishuqin.cn/Snipaste_2023-03-04_10-11-06.png)

勾选WebDAV选项
![](https://cdn.jishuqin.cn/202303041014548.png)

返回主页，点击左侧连接，创建一个WebDAV账号。

![](https://cdn.jishuqin.cn//202303041028287.png)

# 三、windows端

#### 1. 打开windows端的注册表

![](https://cdn.jishuqin.cn//202303041032382.png)

#### 2.访问路径（复制连接粘贴到搜索框按回车即可）：

```
计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters
```

#### 3.把BasicAuthLevel 值改成2，即同时支持http和https，默认只支持https传输协议。

![](https://cdn.jishuqin.cn//202303041037097.png)

#### 4.删除同目录下的 FileSizeLimitInBytes，此操作是为了开放下载大小。

![](https://cdn.jishuqin.cn//202303041039964.png)

#### 5.在当前目录新建 FileSizeLimitInBytes
右键修改 十六进制 <span class="label label-primary">ffffffffffffffff</span>
{% gi 2 2 %}
 ![](https://cdn.jishuqin.cn//202303041044710.png)
 ![](https://cdn.jishuqin.cn/2023-03-03_22-16-16.png)
{% endgi %}

#### 6.打开服务 找到WebClient 
设置为自动启动，并且点击停止 点击启动
{% gi 2 2 %}
 ![](https://cdn.jishuqin.cn//202303041058766.png)
 ![](https://cdn.jishuqin.cn//202303041058742.png)
{% endgi %}
# 四、映射网络驱动器
打开我的电脑，点击映射网络驱动器。
![](https://cdn.jishuqin.cn//202303041101387.png)
填入创建WebDAV账号页面的WebDAV地址。
![](https://cdn.jishuqin.cn//202303041103489.png)
填入账号密码连接到网盘。
![](https://cdn.jishuqin.cn//2023-03-04_11-05-51.png)
查看效果
![](https://cdn.jishuqin.cn//202303041106329.png)

到这Cloudreve映射WebDav就配置完成了，希望对您有所帮助~