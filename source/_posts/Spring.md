title: Travis CI 自动部署Hexo博客至Github
author: Yuanzi
date: 2020-02-15 18:51:07
tags:
---
##### 1.Github配置
Github创建项目xxx.github.io,master为静态页面展示项目，code为hexo代码。
##### 2.Travis CI配置
 用 Github 账号注册并登录 Travis CI，配置好 Travis CI 在 Github 的权限和 Token。
##### 3.在code分支下新建配置文件 .travis.yml
{% codeblock lang:bash %}
sudo: false
language: node_js
node_js:
  - 10 
cache: npm
branches:
    only:
    - code # 当hexo分支有新的commit时执行 
script:
  - hexo generate 
deploy:
    provider: pages
    skip-cleanup: true
    local-dir: public
    target-branch: master # 注意这里是部署到master, 默认是到gh-pages
    github-token: $GH_TOKEN
    keep-history: true
    on:
    branch: code # 博客的源码分支
    

{% endcodeblock %}

##### 4.推送代码至code分支，完成部署

##### 参考教程
https://blog.csdn.net/badcow/article/details/102503102