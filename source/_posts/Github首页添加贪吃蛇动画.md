title: Github首页添加贪吃蛇动画
author: 顾梦
tags:
  - Github
categories:
  - 页面美化
index_img: https://cdn.jishuqin.cn//202306120013028.png
date: 2023-06-11 23:58:00
---

# Github 首页添加贪吃蛇动画

![](https://cdn.jishuqin.cn//snake.svg)

## 一、前言

动画所需要的代码是运行到`Github Actions`。

`GitHub Actions` 是一个用于自动化软件工作流程的平台，包括构建、测试和部署代码。它允许开发人员为软件开发过程的不同阶段创建自定义工作流程，并自动执行重复的任务。`GitHub Actions` 可以集成到 `GitHub` 存储库中，并可以被各种事件（如拉取请求、提交和发布）触发。使用 `GitHub Actions`，开发人员可以轻松协作、减少错误并节省时间。它支持广泛的编程语言，并提供了许多预构建的操作 `actions`在 `GitHub Marketplace` 中使用。

## 二、创建 Action

<p class="note note-warning">需在 Github 账户同名的仓库下创建 Action，点击 New workflow -> Configure。</p>

![](https://cdn.jishuqin.cn//202306120006322.png)

![](https://cdn.jishuqin.cn//202306120006426.png)

点击 Configure 后显示如下界面：

![](https://cdn.jishuqin.cn//202306120007077.png)

## 三、配置脚本

将下面代码复制粘贴到`yml`文件中，然后点击`Commit changes`部署脚本。

```yaml
# GitHub Action for generating a contribution graph with a snake eating your contributions.

name: Generate Snake

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v2.3.4
      
      - name: Generate Snake
        uses: Platane/snk@master
        id: snake-gif
        with:
          github_user_name: ${{ github.repository_owner }}
          gif_out_path: ./assets/github-contribution-grid-snake.gif
          svg_out_path: ./assets/github-contribution-grid-snake.svg

      - name: Push to GitHub
        uses: EndBug/add-and-commit@v7.2.1
        with:
          branch: main
          message: 'Generate Contribution Snake'

```

## 四、设置 workflow 权限

`settings`-> `Actions` -> `General` -> `Read and write permissions`设置读写权限

![](https://cdn.jishuqin.cn//image-20230615223425490.png)

![](https://cdn.jishuqin.cn//image-20230615223547928.png)

## 五、运行 workflow

点击`Actions`到刚才配置好的`workflow`，点击`Run workflow`安全运行脚本。

![](https://cdn.jishuqin.cn//202306120005242.png)

## 六、引入 svg 动画 

```
![](https://raw.githubusercontent.com/你的 GitHub 名/你的 GitHub 名/main/assets/github-contribution-grid-snake.svg)
```

