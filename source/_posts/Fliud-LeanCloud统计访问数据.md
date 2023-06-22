title: Fliud LeanCloud统计访问数据
author: 顾梦
index_img: 'https://cdn.jishuqin.cn//202304091402009.png'
tags:
  - Fluid
categories:
  - 博客配置
date: 2023-04-09 13:59:11
---
## Fluid主题配置 LeanCloud 统计访问数据

### 1.创建LeanCloud实例

<span class="label label-primary">LeanCloud官网</span>：https://console.leancloud.cn/ 

1. 进入官网，注册一个账号

2. 登录LeanCloud 系统点击左上角创建应用

<img src="https://cdn.jishuqin.cn//202304091408027.png" style="zoom:50%;" />

3. 选择开发板创建应用即可

![](https://cdn.jishuqin.cn//202304091409165.png)

3. 创建应用成功后，打开应用设置->应用凭证

   ![](https://cdn.jishuqin.cn//202304091429140.png)

   保存 `AppID`、`AppKey`、`REST API`

### 2.配置_config.fluid.yml文件

1.   在`_config.fluid.yml`文件查找关键字：`web_analytics`

<p class="note note-primary">开启网页访问统计功能</p>

![](https://cdn.jishuqin.cn//202304091420333.png)

2.   在`_config.fluid.yml`文件查找关键字：`leancloud`

<p class="note note-primary">配置 leancloud 数据，将上面的 AppID、AppKey、REST API 填入对应位置</p>

![](https://cdn.jishuqin.cn//202304091433105.png)

3.   在`_config.fluid.yml`文件查找关键字：`views`

<p class="note note-primary">开启浏览量计数</p>

![](https://cdn.jishuqin.cn//202304091434384.png)

4.  在`_config.fluid.yml`文件查找关键字：` leancloud`

<p class="note note-primary">设置 PV、UV 统计数</p>

![](https://cdn.jishuqin.cn//202304091436278.png)

5. 重新部署 Hexo

```
# 清除缓存
hexo clean

# 生成静态资源
hexo g

# 部署
hexo d
```

`hexo g` 是 `hexo generate` 的简写；`hexo d` 是 `hexo deploy` 的简写。