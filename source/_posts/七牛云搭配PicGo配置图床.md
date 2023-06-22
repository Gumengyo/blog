title: 七牛云+PicGo搭建图床
tags:
  - 七牛云
categories:
  - 图床
index_img: 'https://cdn.jishuqin.cn/qiniu.png'
author: 顾梦
date: 2023-02-18 10:04:00

---
# 一、概述
<p class="note note-light">在写博文时难免需要插入图片，所有需要有一个存储图片的地方，图片可以存储在本地目录下，但是会占用本地空间，而且图片比较大的话也会影响页面加载，将图片存储在云空间且采用CDN加速则可以很好解决这个问题。下文介绍的是使用七牛云来作为云存储，注册好七牛云账号就有免费10GB标准存储，不过前提需要有一个备案好的域名或者二级域名，配置起来很容易跟着下文一步一步配置就可以了。</p>

![](https://cdn.jishuqin.cn/qiniu1.png)

# 二、七牛云配置

## 1. 注册七牛云账号

登录<a href="https://www.qiniu.com/" title="点击跳转官网"> 七牛云官网</a>，先注册一个七牛云的账号，填写对应信息注册完成后登录账号。

![](https://cdn.jishuqin.cn/20230218102847.png)

绑定邮箱完成认证，才可以领取免费存储和CDN。
{% gi 2 2 %}
 ![](https://cdn.jishuqin.cn/20230218104711.png)
 ![](https://cdn.jishuqin.cn/20230218105013.png)
{% endgi %}

## 2. 创建存储空间
点击对象存储Kodo，进入对象存储界面吗，点击新建空间，输入想要创建空间名称，选择存储区域及访问控制。
![](https://cdn.jishuqin.cn/20230218105918.png)
![](https://cdn.jishuqin.cn/1676689390931.jpg)

<p class="note note-warning">创建完空间之后会分配一个测试域名，测试域名将会在30天后回收，所以需要为空间绑定我们自己的域名。</p>

## 3. 添加自定义域名

点击自定义域名按钮，输入自己的域名（必须是备案过的），其他默认选项就好可以不用修改，点击创建即可。

![](https://cdn.jishuqin.cn/20230218111008.png)



![](https://cdn.jishuqin.cn/20230218111257689.png)

创建好之后需要对七牛创建的加速域名进行 CNAME 配置，且在域名解析添加CNAME记录，下面以华为云为例。

![](https://cdn.jishuqin.cn/20230218111708.png)

![](https://cdn.jishuqin.cn/20230218111951.png)

添加完成后刷新可以看到CANME状态显示已配置说明域名配置好了，可以看到空间已经绑定上了域名。

![](https://cdn.jishuqin.cn/20230218112204227.png)

![](https://cdn.jishuqin.cn/20230218112446.png)

## 4. 创建密钥

点击密钥管理，创建一个密钥，配置PicGo的使用需要用到。

![](https://cdn.jishuqin.cn/20230218113445.png)

# 三、PicGO

## 1. 下载PicGo并安装

PicGo官网：https://github.com/Molunerfinn/PicGo/releases

红框为win版，蓝框为Mac版

![](https://cdn.jishuqin.cn/20230218114229.png)

## 2. 配置PicGo

安装好之后打开PicGo，找到图床设置，配置七牛云存储。

![](https://cdn.jishuqin.cn/20230218114444.png)

查询七牛云存储空间的区域码，我上边存储区域设置的是华南-广东所以填区域码z2。 
链接：https://developer.qiniu.com/kodo/1671/region-endpoint-fq

![](https://cdn.jishuqin.cn/20230218114956.png)

配置好之后选择图床并且设置为默认图床

![](https://cdn.jishuqin.cn/20230218115508.png)

配置完之后可以上传图片测试一下，PicGo的各项操作可以查询官方文档：https://picgo.github.io/PicGo-Doc/zh/guide/#picgo-is-here

以上就是配置七牛云配置PicGo的步骤啦，希望可以帮到你。😁