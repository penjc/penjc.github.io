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

### 修改历史提交记录
```bash
git filter-branch -f --env-filter '
OLD_EMAIL="2686728826@qq.com"
CORRECT_NAME="PENG"
CORRECT_EMAIL="jcpeng3-c@my.cityu.edu.hk"
if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
export GIT_COMMITTER_NAME="$CORRECT_NAME"
export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
export GIT_AUTHOR_NAME="$CORRECT_NAME"
export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```
### 常见的提交类型
在代码提交中，常见的提交类型（如 feature 和 refactor）通常是基于 Conventional Commits 规范的。

#### feat (feature)

表示新增功能。
例如：feat: 添加用户登录功能
#### fix

表示修复 bug。
例如：fix: 修复无法保存用户数据的问题
#### refactor

表示代码重构，既不影响功能，也不修复 bug。
例如：refactor: 优化查询逻辑以提高性能
#### docs

表示仅修改文档。
例如：docs: 更新 README 文件中的安装指南
#### style

表示代码格式化修改，与功能无关（例如空格、缩进、分号等）。
例如：style: 调整代码缩进以符合规范
#### test

表示添加或修改测试。
例如：test: 添加单元测试用例以覆盖 edge case
#### chore

表示杂项更新，与代码运行逻辑无关（如构建工具配置、CI/CD 配置）。
例如：chore: 更新 npm 依赖版本
#### perf

表示性能优化。
例如：perf: 优化循环处理逻辑以降低时间复杂度
#### build

表示构建系统或外部依赖的修改（如升级 npm 包、修改 Webpack 配置）。
例如：build: 升级 Babel 版本到 v7.20
#### ci

表示与持续集成相关的更改。
例如：ci: 修改 GitHub Actions 配置以支持多平台测试
#### revert

表示回滚之前的提交。
例如：revert: 回滚 "feat: 添加支付功能"
#### hotfix

表示紧急修复问题（不是官方规范的一部分，但常用）。
例如：hotfix: 修复生产环境崩溃问题

---

# **Nginx 配置与网站自动更新总结**

## **1. Nginx 站点配置文件 `/etc/nginx/sites-available/cityu-website`**
### **配置 HTTPS 及自动跳转**
编辑 Nginx 站点配置：
```bash
sudo nano /etc/nginx/sites-available/cityu-website
```

#### **配置内容**
```nginx
server {
    listen 80;
    server_name _;

    # HTTP 重定向到 HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name _;

    root /var/www/cityu-website;
    index index.html;

    # SSL 证书路径
    ssl_certificate /etc/nginx/ssl/stepforx.com_bundle.crt;
    ssl_certificate_key /etc/nginx/ssl/stepforx.com.key;

    ssl_session_timeout 5m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 网站路径
    location / {
        alias /var/www/cityu-website/;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # 其他路径
    location / {
        try_files $uri $uri/ =404;
    }
}
```

**启用配置**：
```bash
sudo ln -s /etc/nginx/sites-available/cityu-website /etc/nginx/sites-enabled/
```


## **2. 安装 SSL 证书**
1. **上传 SSL 证书**：
   - 将 `stepforx.com_bundle.crt` 和 `stepforx.com.key` 上传到 `/etc/nginx/ssl/`
   - 确保目录存在：
     ```bash
     sudo mkdir -p /etc/nginx/ssl
     sudo mv stepforx.com_bundle.crt stepforx.com.key /etc/nginx/ssl/
     ```

2. **验证 Nginx 配置**：
   ```bash
   sudo nginx -t
   ```

3. **重启 Nginx**：
   ```bash
   sudo systemctl reload nginx
   ```


## **3. Nginx 全局配置 `/etc/nginx/nginx.conf`**
编辑主配置文件：
```bash
sudo nano /etc/nginx/nginx.conf
```

**确保包含以下内容**：
```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    access_log /var/log/nginx/access.log;

    gzip on;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    include /etc/nginx/sites-enabled/*;
}
```

**应用更改**：
```bash
sudo nginx -t
sudo systemctl reload nginx
```

## **4. 更新 `build` 文件夹**
每次更新 `build` 文件后，需要执行：
```bash
sudo rm -rf /var/www/cityu-website/*
sudo cp -r /path/to/new-build/* /var/www/cityu-website/
sudo chown -R www-data:www-data /var/www/cityu-website
sudo chmod -R 755 /var/www/cityu-website
sudo nginx -t
sudo systemctl reload nginx
```

## **5. 自动更新脚本**
创建自动更新脚本 `/usr/local/bin/update-website.sh`：
```bash
sudo nano /usr/local/bin/update-website.sh
```

**脚本内容**：
```bash
#!/bin/bash

TARGET_DIR="/var/www/cityu-website"
REPO_URL="git@gitee.com:penjc/cityu-build.git"
TEMP_DIR="/tmp/cityu-build"

# 拉取最新代码
if [ -d "$TEMP_DIR" ]; then
    rm -rf "$TEMP_DIR"
fi

git clone "$REPO_URL" "$TEMP_DIR"

# 覆盖目标目录
rm -rf "$TARGET_DIR/*"
cp -r "$TEMP_DIR/"* "$TARGET_DIR/"

# 设置权限
chown -R www-data:www-data "$TARGET_DIR"
chmod -R 755 "$TARGET_DIR"

# 重载 Nginx
systemctl reload nginx

# 清理临时目录
rm -rf "$TEMP_DIR"
```

**设置可执行权限**：
```bash
sudo chmod +x /usr/local/bin/update-website.sh
```


## **6. 定时任务（每天凌晨 1 点自动更新）**
使用 `crontab` 设置定时任务：
```bash
sudo crontab -e
```

添加以下内容：
```bash
0 1 * * * /usr/local/bin/update-website.sh >> /var/log/update-website.log 2>&1
```

**测试任务是否生效**：
```bash
sudo crontab -l
```

**手动运行测试**：
```bash
sudo /usr/local/bin/update-website.sh
```

### **7. 任务总结**
| 任务 | 命令 |
|------|------|
| **验证 Nginx 配置** | `sudo nginx -t` |
| **重载 Nginx** | `sudo systemctl reload nginx` |
| **更新 `build` 文件夹** | `sudo rm -rf /var/www/cityu-website/* && sudo cp -r /path/to/new-build/* /var/www/cityu-website/` |
| **设置权限** | `sudo chown -R www-data:www-data /var/www/cityu-website && sudo chmod -R 755 /var/www/cityu-website` |
| **手动运行更新脚本** | `sudo /usr/local/bin/update-website.sh` |
| **查看定时任务** | `sudo crontab -l` |


### **最终效果**
- Nginx 配置了 **HTTPS** 并支持 **自动跳转 HTTP → HTTPS**。
- 服务器每天 **凌晨 1 点自动从 Gitee 拉取代码并部署**。
- 所有更新会自动 **应用权限** 并 **重载 Nginx**。



---

