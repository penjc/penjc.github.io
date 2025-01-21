---
title: Example
excerpt: Hello World
date: 2024-10-10 10:00:00
updated: 2024-10-10 10:00:00
hide: false
archive: false
comments: true
index_img: https://images.unsplash.com/photo-1495106245177-55dc6f43e83f?q=80&w=2970&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
#tags: 
#  - Hello World
#categories:
#  - [Sports, Baseball]
#  - [MLB, American League, Boston Red Sox]
#  - [MLB, American League, New York Yankees]
#  - Rivalries
#sticky: 100 数字越大越靠前
---


Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

### Tag

{% note success %}
primary、secondary、success、danger、warning、info、light
{% endnote %}

### 折叠块

{% fold primary @折叠块 %}
primary、secondary、success、danger、warning、info、light
{% endfold %}

### 勾选框

[//]: # ({% cb 勾选, checked?, incline? %})
[//]: # (checked? 为 true 时，勾选框默认勾选)
[//]: # (incline？为 true，后面的文字不换行)
{% cb 勾选, checked?, false %}

### 跳转按钮

{% btn url, 跳转按钮, 悬停文字 %}

### 图片集

[//]: # ({% gi 5 2-2-1 %} 5行2列，2:2:1比例)
[//]: # (![](图片地址) 图片地址)
[//]: # ({% endgi %})
{% gi 5 2-2-1 %}
![](https://images.unsplash.com/photo-1494199505258-5f95387f933c?q=80&w=2973&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D)
![](https://images.unsplash.com/photo-1451187580459-43490279c0fa?q=80&w=2972&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D)
![](https://images.unsplash.com/photo-1435783099294-283725c37230?q=80&w=3028&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D)
![](https://images.unsplash.com/photo-1508201890890-b6dec8b3fb2a?q=80&w=2972&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D)
![](https://images.unsplash.com/photo-1495106245177-55dc6f43e83f?q=80&w=2970&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D)
{% endgi %}

### 总定义页面

1.首先用命令行创建页面：
```
hexo new page example
```
2.创建成功后编辑博客目录下 /source/example/index.md：

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
