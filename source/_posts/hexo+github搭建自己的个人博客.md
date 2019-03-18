---
title: hexo+github搭建自己的个人博客
date: 2019-03-15 09:48:37
tags: hexo
---

## 安装相关依赖
- 安装node(mac为例)  
```
brew install node
node -v (是否安装成功)
```
- 安装git  
```
brew install git
git -v
```
- 安装hexo  
```
npm install hexo-cli -g
hexo -v
```
如果上述步骤都没有问题，说明本地配置已经搞定，然后就可以在github上搭建项目了。

## github上创建并设置仓库
新建仓库，在repository下填写你要创建的地址，这个地址就是你的域名，以我仓库为例，fkrin.github.io，***一定要以xxx.github.io为结尾命名项目***。创建完之后，点击setting设置仓库，找到Github Pages点击change theme选择主题并设置模版。

## 初始化hexo
- 首先将项目clone下来  
`https://github.com/FKIRIN/fkirin.github.io.git`
- 然后cd进入到项目根目录  
`cd fkirin.github.io`
- 然后进行hexo初始化的相关命令
```
npm install hexo --save
hexo init
npm install
hexo g
hexo s(启动项目，默认在4000端口)
```

- 在地址栏中输入localhost：4000可以看到这样的默认页面![显示](/images/2019-3/2019-03-19.jpg)

- 使用其他主题（以next为例）
`git clone https://github.com/iissnan/hexo-theme-next themes/next`
然后配置_config.yml,将theme改为使用next，注意:有个空格![显示](/images/2019-3/2019-03-19-1.jpg)，
其他配置：  

  1. title: 网站标题
  2. subtitle: 副标题
  3. description: 个人签名
  4. author: 姓名
  5. language: zh-Hans
配置deploy的参数，参考上图。不能遗漏。

## 提交项目部署
- 如果想要新增文章需要使用
`hexo new "名称"`来新增md文件，否则无法识别。
- 安装部署工具
`npm install hexo-deployer-git --save`
- 移除子模块  
因为next主题是其他仓库的内容，相当于本仓库的一个子模块，所以需要删除这个子模块将其当作此项目的一部分内容。
```
step1: 
$ git rm --cached themes/next
rm 'themes/next'
step2: 
$ git add themes/next/
注意： 这里next之后一定要加上/，表示将这个文件夹加入，而不是将这个文件夹当作一个子模块。
step3: 
$ git commit -m "提交信息"
$ git push origin master 
step4: 
$ hexo g （刷新）
$ hexo d  （部署）
（如果不执行这两个命令，网页上的内容不会更新)
```
参考： [仓库内克隆其他仓库遇到的问题](https://upupming.site/2018/05/31/git-submodules/#%E4%BB%93%E5%BA%93%E5%86%85%E5%85%8B%E9%9A%86%E5%85%B6%E4%BB%96%E4%BB%93%E5%BA%93%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98)  
此处是直接以master分支开发的，具体可以新起一个dev分支，通过merge dev分支来更新master上的内容。
