---
title: Hexo博客快速部署
date: 2019-07-01 23:29:08
tags: [javascript, hexo]
---

## 准备工作

- 安装Git, 我使用的是图形化Git工具[git-fork](https://git-fork.com/)的内嵌Git. 注意把`git.exe`所在目录添加到环境变量, 安装建议前先测试git环境.

- 安装[Node.js](https://nodejs.org/en/), 推荐安装LTS版.

- 安装npm(Node.js包管理工具)的国内镜像版cnpm:

    ```bash
    npm install -g cnpm --registry=https://registry.npm.taobao.org
    ```

- 安装hexo:

    ```bash
    cnpm install -g hexo-cli
    ```
<!-- more -->
## 配置Hexo

```bash
# 创建一个空文件夹并初始化hexo
mkdir "Hexo"
cd Hexo
hexo init
# 启动服务器, 预览用
hexo s    # hexo server
# 创建文章
hexo n "<sample-article.md>"    #hexo new "<sample-article.md>"
# 进行清理
hexo clean
# 生成新的页面
hexo g    # hexo generate
```

安装`hexo-deployer-git`工具方便部署到Github Pages.

```bash
cnpm install --save hexo-deployer-git
```

## 部署Hexo

1. 首先创建仓库

    仓库名称必须命名为`GitHub用户名.github.io`!

2. 其次修改Hexo根目录的配置文件`_config.yml`

    ```yml
    # Deployment
    ## Docs: https://hexo.io/docs/deployment.html
    deploy:
        type: git
        repo: https://github.com/你的GitHub用户名/你的GitHub用户名.github.io.git
        branch: master
    ```

3. 之后进行部署

    ```bash
    hexo d    # hexo deploy
    ```

## 更换主题

[yilia](https://github.com/litten/hexo-theme-yilia)是一款很简约且适应移动端的主题, 文档已经写得非常不错, 这里就不赘述了.

## Hexo更换部署电脑

首先要求Hexo目录本身是利用Git分支同步的(我是master分支用来部署网站, hexo分支存放源文件).

首先在新的机器上`hexo init <dir-name>`, 然后等Hexo配置完, **删掉**除了`node_modules`和`public`文件夹之外的所有文件, 之后`git init`, `git add remote`之后把Github的hexo分支拉取下来, 这样就没有问题了.

然后要再安装`cnpm install --save hexo-deployer-git`, 即配置完毕.

原理: `node_modules`是Hexo依赖的模块, 各电脑不一样, 所以要在新机器上`hexo init`手动配置. `public`类似于临时文件, 也不需要通过Git拉取. 以上两个文件夹都是写在`.gitignore`中的.
