title: Cloudreve部署配置
author: 顾梦
tags:
  - Cloudreve
index_img: 'https://cdn.jishuqin.cn/cloudreve.png'
categories:
  - 服务配置
date: 2023-03-02 11:21:00
---
# 一、前言

Cloudreve 可以让您快速搭建起公私兼备的网盘系统。Cloudreve 在底层支持不同的云存储平台，用户在实际使用时无须关心物理存储方式。你可以使用 Cloudreve 搭建个人用网盘、文件分享系统，亦或是针对大小团体的公有云系统。拥有一个自己的网盘，保存文件下载文件将非常方便。

![](https://cdn.jishuqin.cn/2023-03-03_21-10-31.png)

# 二、安装Cloudreve

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

<p class="note note-warning">注意Cloudreve首次启动时，会创建初始管理员账号和密码，只有首次启动时才会出现，所以这里需要保存好账号密码，后边登录系统时使用。</p>

# 三、进程守护
可以使用Systemd或Supervisor守护进程，下边介绍使用Systemd，可以参考[官方文档](https://docs.cloudreve.org/getting-started/install#jin-cheng-shou-hu)。

```
# 编辑配置文件
vim /usr/lib/systemd/system/cloudreve.service
```
将下文 PATH_TO_CLOUDREVE 更换为程序所在目录：

```
[Unit]
Description=Cloudreve
Documentation=https://docs.cloudreve.org
After=network.target
After=mysqld.service
Wants=network.target

[Service]
WorkingDirectory=/PATH_TO_CLOUDREVE
ExecStart=/PATH_TO_CLOUDREVE/cloudreve
Restart=on-abnormal
RestartSec=5s
KillMode=mixed

StandardOutput=null
StandardError=syslog

[Install]
WantedBy=multi-user.target
```
更新配置
```
# 更新配置
systemctl daemon-reload

# 启动服务
systemctl start cloudreve

# 设置开机启动
systemctl enable cloudreve
```

管理命令：

```
# 启动服务
systemctl start cloudreve

# 停止服务
systemctl stop cloudreve

# 重启服务
systemctl restart cloudreve

# 查看状态
systemctl status cloudreve
```

# 四、使用Nginx Proxy Manager反向代理

配置反向代理需要有一个自己的域名，并解析到启动Cloudreve的服务器。
{% gi 2 2 %}
 ![](https://cdn.jishuqin.cn/2023-03-03_22-11-03.png)
 ![](https://cdn.jishuqin.cn/2023-03-03_22-16-16.png)
{% endgi %}

# 五、系统配置

1. 访问上边配置反向代理的域名进入系统登录页面，使用首次启动系统生成的账户名和密码登录系统。

![](https://cdn.jishuqin.cn/cloudreve4.png)

2. 点击用户头像，进入管理面板。
![](https://cdn.jishuqin.cn/cloudreve5.png)
3. 点击用户界面，设置你自己的账号密码。
![](https://cdn.jishuqin.cn/cloudreve6.png)
<p class="note note-info">默认存储策略是保存到系统本地，想配置七牛云存储、腾讯云COS等云存储，点击添加存储策略，选择对应的存储服务会有详细的配置说明。</p>

![](https://cdn.jishuqin.cn/cloudreve7.png)

到这cloudreve基本搭建完成啦，试试上传文件到网盘吧~