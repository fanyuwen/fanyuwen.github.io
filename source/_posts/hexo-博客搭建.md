---
title: hexo 博客搭建
date: 2021-02-24 23:00:18
tags:
---

1. 先安装node(包括npm),确保已安装执行node -v和npm -v命令
2. 运行npm install hexo命令安装hexo
3. 新建自己的博客目录,在该目录下运行hexo init命令
4. 初始化好了之后运行hexo s命令并打开浏览器访问localhost:4000,查看默认生成的博客
5. 去自己的github上新增一个仓库,仓库名必须是 用户名.github.io
6. 
7. 需要用npm下载hexo-deployer-git npm install --save hexo-deployer-git
8. 去自己的博客目录下打开_config.yml文件,去最下面的deply配置
9. 修改并添加如下
```yaml
deploy:
  type: git
  repo: 仓库地址
  branch: 分支名
```
10. 可以运行hexo d将生成的文件推送到远程git仓库,可以请求 用户名.github.io 看已经生成的博客
11. 可以使用hexo new [(可选)post] "标题名" 创建一个新标题
12. 编辑完标题之后,运行hexo clean(清理) 然后再运行hexo g(生成) 也可以试着运行hexo s 本地查看是否正确
13. 正确无误运行hexo d推送到远程仓库,查看最新的博客