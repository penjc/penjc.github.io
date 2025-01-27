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

### nginx 操作
```
sudo chown -R www-data:www-data /var/www/cityu-website
sudo chmod -R 755 /var/www/cityu-website
```

### 重启 nginx
```bash
sudo systemctl reload nginx
```

可以通过 **crontab** 设置定时任务，每天凌晨 1 点从 Gitee 仓库拉取代码，覆盖指定目录，然后执行权限设置和重载 Nginx 的操作。

以下是完整的实现步骤：

---
## 🕒 定时拉取代码并部署
### **1. 确保 Gitee 仓库的 SSH 访问已配置（推荐）**
为了避免每次拉取代码需要输入密码，建议配置 Gitee 的 SSH 密钥。

#### **1.1 生成 SSH 密钥**
如果服务器还没有 SSH 密钥，可以生成一个：
```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```
按提示操作，生成的密钥文件默认位于 `~/.ssh/id_rsa`。

#### **1.2 添加公钥到 Gitee**
1. 打开 `~/.ssh/id_rsa.pub`，复制其中的内容：
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

2. 登录 Gitee，前往 **"账号设置" > "SSH公钥"**，将公钥添加到你的账号中。

#### **1.3 测试 SSH 连接**
执行以下命令，确保可以通过 SSH 连接到 Gitee：
```bash
ssh -T git@gitee.com
```
如果成功，会显示类似：
```
Welcome to Gitee.com!
```

---

### **2. 创建拉取代码并部署的脚本**
创建一个脚本，用于从 Gitee 拉取代码、覆盖目标目录并设置权限。

#### **脚本路径**：
假设脚本存放在 `/usr/local/bin/update-website.sh`。

#### **脚本内容**：
```bash
#!/bin/bash

# 目标目录和仓库地址
TARGET_DIR="/var/www/cityu-website"
REPO_URL="git@gitee.com:penjc/cityu-build.git"

# 临时目录
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

#### **设置脚本权限**：
```bash
sudo chmod +x /usr/local/bin/update-website.sh
```

---

### **3. 添加定时任务**
使用 `crontab` 设置每天凌晨 1 点执行脚本。

#### **编辑定时任务**：
打开 `crontab` 配置：
```bash
sudo crontab -e
```

#### **添加以下内容**：
```bash
0 1 * * * /usr/local/bin/update-website.sh >> /var/log/update-website.log 2>&1
```

**解释**：
- `0 1 * * *`：每天凌晨 1 点执行。
- `>> /var/log/update-website.log 2>&1`：将脚本输出和错误日志记录到 `/var/log/update-website.log`。

---

### **4. 验证定时任务**
#### **手动运行脚本测试**：
在配置完成后，手动运行脚本以确保其工作正常：
```bash
sudo /usr/local/bin/update-website.sh
```

#### **检查日志**：
如果没有错误，可以查看 `/var/log/update-website.log` 确认是否记录成功。

#### **查看定时任务是否生效**：
使用以下命令确认定时任务是否已加载：
```bash
sudo crontab -l
```

---

### **总结**
设置完成后，服务器将每天凌晨 1 点：
1. 从 Gitee 仓库拉取最新代码。
2. 覆盖到 `/var/www/cityu-website`。
3. 设置权限并重载 Nginx。
