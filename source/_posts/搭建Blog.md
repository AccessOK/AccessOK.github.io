---
title: 搭建Blog
date: 2023-12-14 10:27:28
description: 梳理Blog搭建流程
type: "tags"
comments: true
categories:
- Learning Tools
- Hexo
tags:
- nodejs
- Blog
---

# 搭建Blog

## 安装node npm n

- 安装node.js

  ```bash
  sudo apt-get install nodejs
  ```

- 安装npm
  npm是nodejs的包管理器。
  
  ```bash
  sudo apt-get install npm
  ```

- 安装n

  n 是交互式 node.js 版本管理工具。

  ```bash
  sudo npm install -g n
  ```

  可以使用n安装特定版本的nodejs。

## 安装hexo

​Hexo 是**一个快速、简洁且高效的博客框架**。 Hexo 使用Markdown（或其他标记语言）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

- 配置国内源

  ```bash
  sudo npm set registry=https://registry.npm.taobao.org
  ```

- 安装hexo-cli

  ``` bash
  sudo npm install hexo-cli -g
  ```

- 初始化项目

  ```bash
  sudo npm init blog
  ```

- 安装依赖

  ``` bash
  npm install
  ```

- 打包成dist

  ``` bash
  npm run build
  ```
  
- 运行hexo项目

  ```bash
  hexo server
  ```

- 清楚hexo部署数据

  ```bash
  #可删除.deploy_git文件夹在执行hexo clean和hexo deploy
  hexo clean
  ```

- 部署到服务器

  ```bash
  #根据_config.yaml配置文件中的仓库地址，部署到服务器。
  hexo delpoy
  ```

- 替换主题

  ```bash
  安装主题和渲染器：
  $ git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant  
  $ npm install hexo-renderer-pug --save  
  $ npm install hexo-renderer-sass --save  
  编辑Hexo目录下的 _config.yml，将theme的值改为maupassant。
  注：依赖hexo-renderer-sass安装时容易报错，很可能是国内网络问题，请尝试使用代理或者切换至NPM的国内镜像源安装。
  ```

- 创建"About页"

```bash
hexo new page about
创建成功后，编辑博客目录下 /source/about/index.md，添加 layout 属性。
```

- 创建"Archive页"

```bash
hexo new page archive
创建成功后，编辑博客目录下 /source/archive/index.md。
```

- hexo使用参考：<https://hexo.io/zh-cn/>
