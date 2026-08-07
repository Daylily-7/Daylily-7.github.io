+++

date = '2026-07-29T16:26:31+08:00'

draft = false

title = '从零搭建个人博客：我的 Hugo + GitHub Pages 折腾笔记'

+++

+++
date = '2026-07-29T16:26:31+08:00'
draft = false
title = '从零搭建个人博客：我的 Hugo + GitHub Pages 折腾笔记'
+++

从零搭建个人博客：我的 Hugo + GitHub Pages 折腾笔记



2026年7月28和29日，我花了两天时间，从完全不懂到拥有一个属于自己的个人博客

这篇文章用以记录我遇到的坑和解决办法


一．准备工作：安装工具

1.安装 Hugo（静态网站生成器）
Hugo 的作用：把我写的 Markdown 文章转换成 HTML 网页
我用 Windows 的包管理器 Scoop 安装
1.安装 Scoop
打开一个普通权限的 PowerShell 窗口（注意：不要用管理员权限），依次执行：

```
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

irm get.scoop.sh | iex
```

2.安装 Hugo：

```
scoop install hugo-extended
```

3.验证是否安装成功：

```
hugo version
```

如果看到版本号，说明安装成功了
（我第一次用管理员权限运行 PowerShell 安装 Scoop 失败了，提示"禁止以管理员身份运行"，解决方法是关掉管理员窗口，用普通权限的 PowerShell 重新执行）

2. 安装 Git
Git 用来把代码推送到 GitHub。
1.安装 Git：

```
scoop install git
```

2.验证是否安装成功：

```
git --version
```

二、创建博客站点
1. 生成站点文件夹
执行以下命令：

```
cd \~\\Desktop
hugo new site my-blog
cd my-blog
git init
```

这时我的桌面多了一个 my-blog 文件夹，里面是博客的骨架。

2. 添加主题
Hugo 本身不带样式，需要安装主题（我选择了简洁的PaperMod
我一开始用的命令是：

```
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

但我了解到国内访问 GitHub 经常超时，子模块下载失败，试了好几遍依然不行
我最后直接从浏览器下载主题的 ZIP 包，手动解压到 themes/PaperMod 文件夹——

```
手动下载的步骤：

1.打开浏览器，访问 https://github.com/adityatelange/hugo-PaperMod/archive/refs/heads/master.zip

2.下载完成后解压，得到 hugo-PaperMod-master 文件夹

3.把文件夹重命名为 PaperMod

4.把 PaperMod 文件夹复制到 my-blog/themes/ 里面
```

3. 配置博客基本信息
用记事本打开 hugo.toml：

```
notepad hugo.toml
```

写入以下内容（记得把用户名换成自己的）：

```
baseURL = "https://用户名.github.io/"
title = "我的个人博客"
theme = "PaperMod"
languageCode = "zh-cn"

\[params]
&#x20;   description = "这是我的个人博客"
&#x20;   author = "用户名"
&#x20;   hideMeta = true

\[\[menu.main]]
&#x20;   name = "首页"
&#x20;   url = "/"
&#x20;   weight = 1
```

*关于 hideMeta：这个是我后来加的，为了让博客更简洁，不显示每篇文章的日期和作者。

三、写第一篇文章

1.创建文章：

```
hugo new content posts/我的第一篇文章.md
```

2.用记事本编辑：

```
notepad content/posts/我的第一篇文章.md
```

打开后把 draft: true 改成 draft: false（这样才能发布），然后在 --- 下方写正文：

```
+++

title= "我的第一篇文章"
date=2026-07-28
draft= false
+++

Hello World！梦从这里开始——
```

3.本地预览：

```
hugo server -D
```

打开浏览器访问 http://localhost:1313/，就能实时看到博客的样子，修改文章内容，页面会自动刷新（这个功能很方便

四、部署到 GitHub Pages（这一步差点给我整力竭了
1. 在 GitHub 创建仓库

```
1.登录 GitHub，点击右上角的加号，选择 New repository
2.仓库名填写：你的用户名.github.io（比如我的 Daylily-7.github.io）
3.选择 Public
4.不要勾选 "Add a README file"
5.点击 Create repository
```

2. 关联本地仓库并推送

```
git remote add origin https://github.com/用户名/用户名.github.io.git
git add .
git commit -m "first commit"
git branch -M main
git push -u origin main
```

*我遇到的坑：

推送时提示 error: src refspec main does not match any，原因是 Git 的用户信息没配置

通过执行以下命令可以解决：

```
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

*还有一个坑——推送时网络超时：
GitHub在国内访问不稳定，我换了 SSH 方式推送，但是SSH 也超时
最终改用 HTTPS + Token 方式才成功

*生成 Token 的步骤：

```
1.GitHub 右上角头像 -> Settings -> Developer settings -> Personal access tokens -> Tokens (classic)
2.点击 Generate new token (classic)
3.Note 填写一个名字，Expiration 选 No expiration，Scopes 勾选 repo
4.点击 Generate token，复制生成的 Token
```

推送时把Token粘上去

3.配置 GitHub Pages 和 GitHub Actions
为了让博客自动构建和部署，我用了 GitHub Actions
1.创建 GitHub Actions 配置文件：

```
mkdir .github\\workflows
notepad .github\\workflows\\gh-pages.yml
```

2.粘贴以下内容：

```
name: Deploy Hugo Site

on:
&#x20; push:
&#x20;   branches:
&#x20;     - main

jobs:
&#x20; build-deploy:
&#x20;   runs-on: ubuntu-latest
&#x20;   steps:
&#x20;     - uses: actions/checkout@v4
&#x20;       with:
&#x20;         submodules: true

&#x20;     - name: Setup Hugo
&#x20;       uses: peaceiris/actions-hugo@v2
&#x20;       with:
&#x20;         hugo-version: '0.164.0'
&#x20;         extended: true

&#x20;     - name: Build
&#x20;       run: hugo --minify

&#x20;     - name: Deploy to GitHub Pages
&#x20;       uses: peaceiris/actions-gh-pages@v3
&#x20;       with:
&#x20;         github\_token: ${{ secrets.GITHUB\_TOKEN }}
&#x20;         publish\_dir: ./public
&#x20;         publish\_branch: gh-pages
```

3.删除本地的 public 文件夹：

```
Remove-Item -Recurse -Force public
```

4.推送代码：

```
git add .
git commit -m "添加 GitHub Actions 自动部署"
git push
```


5.在 GitHub 上设置 Pages：
进入仓库的 Settings -> Pages，Source 选择 Deploy from a branch，Branch 选择 gh-pages，文件夹选 / (root)，点击 Save。


五、我的踩坑记录

1.SSH 连接超时：
我一开始尝试用 SSH 推送时，ssh -T git@github.com 一直超时
解决方法：改用 HTTPS + Personal Access Token

2.主题文件丢失：
在 main 分支和 gh-pages 分支之间切换时，主题文件经常丢失，themes/PaperMod 变成空文件夹
解决方法：从 GitHub 手动下载主题 ZIP 包，解压到 themes/PaperMod

3.配置改完后页面没变化
这是浏览器缓存的问题
解决方法：我尝试了用隐私/无痕模式打开博客，按 Ctrl+F5 强制刷新了好几次，最后清除掉浏览器缓存后就能正常打开了


六、总结每次成功的部署流程：
1.生成网站文件：

```
hugo --minify --cleanDestinationDir
```

2.切换到 gh-pages 分支：

```
git checkout gh-pages --force
```

3.清理 gh-pages 分支：

```
git rm -rf .
```

4.复制新生成的 public 文件：

```
Copy-Item -Path .\\public\\\* -Destination . -Recurse -Force
```

5.添加所有文件：

```
git add .
```

6.提交：

```
git commit -m "更新博客"
```

7.推送到 GitHub：

```
git push origin gh-pages
```

8.切回主分支：

```
git checkout main
```


七、用AI总结的常用命令速查

```
生成网站：hugo --minify --cleanDestinationDir
本地预览：hugo server -D
创建文章：hugo new content posts/文章名.md
部署第一步：git checkout gh-pages --force
部署第二步：git rm -rf .
部署第三步：Copy-Item -Path .\\public\\\* -Destination . -Recurse -Force
部署第四步：git add .
部署第五步：git commit -m "更新博客"
部署第六步：git push origin gh-pages
部署第七步：git checkout main
```
