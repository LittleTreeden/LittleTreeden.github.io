---
title: hexo-Deploy
date: 2022-07-21 23:16:27
tags:
---
# 使用Travis CI 实现 Hexo 博客自动部署

## 1. 下载工具[Node.js](https://nodejs.org/en/) 和 [Git](https://git-scm.com/)

-   下载完成后使用 `node -v` 检测是node否安装完成

![image-20220721131809145](image-20220721131809145.png)

-   `git --version` 检测git是否安装完成

![image-20220721132031956](image-20220721132031956.png)

## 2.新建Git仓库 `username.github.io`

-   在右上角点击新建仓库。

![image-20220721132450874](image-20220721132450874.png)

-   仓库名使用 `username.github.io`，点击创建。

>   <font color = "#FF6347">username</font> 为 github 账户名

![image-20220721132729556](image-20220721132729556.png)

## 3.将仓库clone到本地

![image-20220721191631507](image-20220721191631507.png)

## 4. 下载并安装hexo



1.   安装hexo脚手架 `npm i hexo-cli -g`

![image-20220721191925191](image-20220721191925191.png)

检查hexo是否安装完成 `hexo -v`

![image-20220721192511927](image-20220721192511927.png)

2.   新建分支 `hexo` 

![image-20220721192757289](image-20220721192757289.png)

3.   新建一个空文件夹，并初始化hexo `hexo init`

![image-20220721192955232](image-20220721192955232.png)

4.   使用npm安装依赖库`npm install`

![image-20220721193036774](image-20220721193036774.png)

5.   使用hexo生成静态网页 `hexo g`

![image-20220721193116293](image-20220721193116293.png)

6.   运行hexo `hexo s`

![image-20220721193204043](image-20220721193204043.png)

​	打开 http://localhost:4000/ 验证

![image-20220721193214091](image-20220721193214091.png)

以上为hexo本地部署

## 5.Travis配置

1.   生成GitHub验证TOKEN

https://github.com/settings/tokens

只选仓库权限即可

![image-20220721195004432](image-20220721195004432.png)

2.   复制token `只会显示一次，记得复制` 

     ![image-20220721195127515](image-20220721195127515.png)

4.   进入[Travis](https://www.travis-ci.com/account/repositories) ，按照提示开启，新注册用户需要选择Plan，需要绑定信用卡。

![image-20220721195600438](image-20220721195600438.png)

5.   对我们的设置相关配置

     ![image-20220721200038829](image-20220721200038829.png)

6.   添加token环境变量，将github生成的token填入，变量名为`GH_TOKEN`

![image-20220721200850986](image-20220721200850986.png)

7.将本地仓库切换至`hexo`，将刚才的博客源码添加到我们的仓库中，并删除`public` 和 `node_modules`

![image-20220721201133162](image-20220721201133162.png)

8.   添加`.travis.yml`文件，并修改相应配置

```yaml
# 指定语言环境
language: node_js
# 指定需要sudo权限
sudo: required
# 指定node_js版本
node_js: v16.2.0
# 指定缓存模块，可选。缓存可加快编译速度。
cache:
  directories:
    - node_modules

# 指定博客源码分支，因人而异。hexo博客源码托管在独立repo则不用设置此项
branches:
  only:
    - hexo 

before_install:
  - npm install -g hexo-cli

# Start: Build Lifecycle
install:
  - npm install
  - npm install hexo-deployer-git --save

# 执行清缓存，生成网页操作
script:
  - hexo clean
  - hexo generate

before_script:
  - git config user.name "lishuyuan"
  - git config user.email "1329838725@qq.com"
  # 替换同目录下的_config.yml文件中github_token字符串为travis后台刚才配置的变量，注>意此处sed命令用了双引号。单引号无效！
  - sed -i "s/github_token/${GH_TOKEN}/g" _config.yml || exit 1

# 设置git提交名，邮箱；替换真实token到_config.yml文件，最后depoy部署
after_script:
  - cat ./_config.yml
  - hexo deploy
# End: Build LifeCycle

# deploy:
#   # provider: pages
#   skip-cleanup: true
#   github-token: $GH_TOKEN
#   keep-history: true
#   on:
#     branch: main
#   local-dir: public
```

9.   修改`_config.yml` 的deploy节点

```yaml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
- type: git
  # 下方的gh_token会被.travis.yml中sed命令替换
  repo: https://github_token@github.com/LittleTreeden/LittleTreeden.github.io.git
  branch: main
```

10.   执行命令 `git push origin hexo` 提交代码

## 相关链接

hexo 官网 https://hexo.io/zh-cn/

参考 https://michael728.github.io/2019/06/16/cicd-hexo-blog-travis/

参考 https://godweiyang.com/2018/04/13/hexo-blog/#toc-heading-8