title: 给 Fluid 主题博客增加打赏功能
author: 顾梦
tags:
  - Fluid
  - Hexo
index_img: 'https://cdn.jishuqin.cn//image-20230618000739725.png'
categories:
  - 博客配置
date: 2023-06-17 22:50:10
---
# 给 Fluid 主题博客增加打赏功能

## 1. 前言

看到别人的博客都有打赏的功能，所以自己也想着在自己博客加入打赏功能，百度到了[谷月](https://blog.kukmoon.com/)的博文，参考博文实现了功能~[^1]

## 2.  覆盖配置

以下配置参考 Hexo Fluid 手册[^2]

<p class="note note-success">TIP:<br>
覆盖配置可以使主题配置放置在 fluid 目录之外，避免在更新主题时丢失自定义的配置。
通过 Npm 安装主题的用户可忽略，其他用户建议学习使用。
</p>

Hexo 5.0.0 版本以上的用户，在博客目录下创建 `_config.fluid.yml` 文件，将主题的[_config.yml](https://github.com/fluid-dev/hexo-theme-fluid/blob/master/_config.yml) 全部配置（或部分配置）复制过去。
以后如果修改任何主题配置，都只需修改 `_config.fluid.yml` 的配置即可。

注意：

- 只要存在于 `_config.fluid.yml` 的配置都是高优先级，修改原 `_config.yml` 是无效的。

- 每次更新主题可能存在配置变更，请注意更新说明，可能需要手动对 `_config.fluid.yml` 同步修改。

- 想查看覆盖配置有没有生效，可以通过 `hexo g --debug` 查看命令行输出。

- 如果想将某些配置覆盖为空，注意不要把主键删掉，不然是无法覆盖的，比如：

  ```yaml
  about:
    icons:  # 不要把 icon 注释掉，否则无法覆盖配置
      # - { class: 'iconfont icon-github-fill', link: 'https://github.com' }
      # - { class: 'iconfont icon-wechat-fill', qrcode: '/img/favicon.png' }
  ```

## 3. 修改主题源文件

![](https://cdn.jishuqin.cn//image-20230617233736066.png)

找到主题源文件`themes/fluid/layout/post.ejs`对应文章模板的 `</div>` 和 `<hr>` 之间，添加以下代码：

```ejs
<!-- 添加打赏模块 -->
<div class="reward-container">
	<% if (theme.donate.enable) { %>
		<button id="rewardBtn" class="reward-btn">
			<% if (config.language == 'zh-CN') { %> 
				❤ 打赏
			<% } else { %>
				Donate
			<% } %>	
		</button>
		<p class="tea">“<%= theme.donate.message %>”</p>
		<div id="rewardImgContainer" class="reward-img-container">
         	<div class="singleImgContainer">
            	<img id="wechatImg" class="reward-img" src="<%= theme.donate.wechatpay %>" alt="微信二维码">
                <p class="wechatPay">微信支付</p>
            </div>
            <div class="singleImgContainer">
               	<img id="alipayImg" class="reward-img" src="<%= theme.donate.alipay %>" alt="支付宝二维码">
                <p class="aliPay">支付宝支付</p>
           	</div>
		</div>
	<% } %>
</div>
```

## 4. 修改主题配置文件

### 4.1 引入 css 和 js 文件

在`themes/fluid/source/css` 创建名为：`reward.css`的文件，并引入以下代码：

```css
.tea {
    font-size: 0.8125em;
    color: #999999;
    margin-top: 10px;
}

.reward-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 50px;
}

.reward-btn {
    padding: 8px 24px;
    font-size: 18px;
    background-color: #007bff;
    color: #fff;
    border: none;
    cursor: pointer;
    border-radius: 10px;
}

.reward-img-container {
    display: none;
    margin-top: 20px;
    /* 图片容器的透明度 */
    opacity: 0;
    /* 过渡效果,使动画更平滑 */
    transition: opacity 2s ease;
}

.reward-img {
    width: 200px;
    margin: 10px;
    border: 1px dashed #ccc;
    border-radius: 4px;
    padding: 10px;
}

/* 单个图片的容器 */
.singleImgContainer {
    width: 50%;
    height: 240px;
}

/* 微信支付和支付宝支付的文字样式 */
.wechatPay,.aliPay {
	text-align: center;
    font-size: 0.8125em;
    color: #999999;
}
```

在`themes/fluid/source/js` 创建名为：`reward.js`的文件，并引入以下代码：

```js
const rewardBtn = document.getElementById('rewardBtn');
const rewardImgContainer = document.getElementById('rewardImgContainer');

if(rewardBtn){
	rewardBtn.onclick = () => {
		rewardImgContainer.style.display = (rewardImgContainer.style.display === 'none' || rewardImgContainer.style.display === '') ? 'inline-flex' : 'none'
		setTimeout(() => {
			rewardImgContainer.style.opacity = (rewardImgContainer.style.opacity === '0' || rewardImgContainer.style.opacity === '') ? '1' : '0'
		}, 10);
	}
}
```

### 4.2 修改 _config.fluid.yml 文件

<p class="note note-warning">如果没有使用覆盖配置，就要打开博客源文件所在的文件夹下的 theme/fluid/_config.yml 文件.</p>

在文件的 `custom_js`和`custom_css`分别引入自定义的 `reward.js`文件和`reward.css`文件。

![](https://cdn.jishuqin.cn//image-20230617235409049.png)

打开博客源文件所在的文件夹下的 `_config.fluid.yml` 文件，在最后添加如下内容：

```yaml
# Donate 自己为 Fluid 主题增加的打赏功能
# `message` 是打赏提示语，可自定义
# `alipay` 是支付宝付款码， `wechatpay` 是微信付款码。
donate:
  enable: true
  message: 觉得不错的话，给点打赏吧 ୧(๑•̀⌄•́๑)૭ 
  alipay: /img/alipay.png
  wechatpay: /img/wechatpay.jpg
```

**参数说明：**

- `message`： 自定义打赏提示语
- `alipay`：支付宝打赏图片
- `wechatpay`：微信打赏图片

**记得设置上自己的打赏图片哟~**

hexo 重新加载配置：

```
hexo clean && hexo g
```

## 5. 效果

![未点击打赏按钮时](https://cdn.jishuqin.cn//image-20230618000812083.png)

![点击打赏按钮时](https://cdn.jishuqin.cn//image-20230618000739725.png)

## 参考

[^1]:[Kukmoon谷月.给 Hexo 博客的 Fluid 主题增加打赏模块.2021-11-18](https://blog.kukmoon.com/7bf76e1d/)
[^2]: [Hexo Fluid 手册](https://hexo.fluid-dev.com/docs/guide/#%E8%A6%86%E7%9B%96%E9%85%8D%E7%BD%AE)