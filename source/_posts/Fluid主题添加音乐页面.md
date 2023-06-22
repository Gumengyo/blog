title: Fluid主题添加音乐功能
author: 顾梦
tags:
  - Fluid
index_img: 'https://cdn.jishuqin.cn//202303051412252.png'
categories:
  - 博客配置
date: 2023-03-05 14:07:00
---
<p class="note note-success">
    本文大部分配置参考林慕凡的博文，感谢作者的详细讲解。<br>
    作者：林慕凡<br>
    原文地址：<a href="https://blog.csdn.net/weixin_43471926/article/details/109798928?spm=1001.2014.3001.5502" target="_blank">https://blog.csdn.net/weixin_43471926/article/details/109798928?spm=1001.2014.3001.5502</a>
</p>

**效果预览：**https://www.jishuqin.cn/playlist/

<img src="https://cdn.jishuqin.cn//202303051412252.png" style="zoom:50%;" />

## 一、安装插件

执行命令，为Hexo安装hexo-tag-aplayer插件。

```
npm install --save hexo-tag-aplayer
```

## 二、添加配置

安装好插件之后，在主题配置文件<span class="label label-primary">_config.fluid.yml </span>中的 custom 添加下面 js 和 css 依赖。

```
custom_js:
  - //cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.js  #/APlayer#/APlayer依赖
  - //cdn.jsdelivr.net/gh/metowolf/MetingJS@1.2/dist/Meting.min.js  #/APlayer依赖
custom_css:
  - //cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.css   #/APlayer依赖

```

添加完成后在hexo根目录下的配置文件<span class="label label-primary">_config.yml </span>中添加以下配置。

```
# aplayer
aplayer:  
  meting: true  
  asset_inject: false
```

## 三、创建音乐页面

```
hexo new page playlist
```

1. 打开 /根目录/source/playlist/index.md编辑页面，参考 hexo-tag-aplayer [官方文档](https://github.com/MoePlayer/hexo-tag-aplayer/blob/master/docs/README-zh_cn.md)书写。
<p class="note note-info">复制歌单的链接，然后复制歌单的id，例如：https://music.163.com/#/playlist?id=8158874729&userid=499215828，这个歌单的id就是8158874729，而且是网易云的歌单，参考下面例子。</p>
```
<!-- 简单示例 (id, server, type)  -->
{% meting "8158874729" "netease" "playlist" %}

<!-- 进阶示例 -->
{% meting "8158874729" "netease" "playlist" "autoplay" "mutex:false" "listmaxheight:340px" "preload:auto" "theme:#3F51B5""order:random"%}
```

2. 下面是有关<span class="label label-info">meting</span>的部分选项配置，具体可参看[官方文档](https://github.com/MoePlayer/hexo-tag-aplayer/blob/master/docs/README-zh_cn.md)。

|  选项  |   默认值   |                           描述                            |
| :----: | :--------: | :-------------------------------------------------------: |
|   id   | **必须值** |       歌曲 id / 播放列表 id / 相册 id / 搜索关键字        |
| server | **必须值** | 音乐平台: `netease`, `tencent`, `kugou`, `xiami`, `baidu` |
|  type  | **必须值** |      `song`,`playlist`, `album`, `search`, `artist`       |

3. 添加音乐页面到导航菜单内。

   ```
     menu:
       - { key: "音乐", link: "/playlist/", icon: "iconfont icon-music" }
   ```