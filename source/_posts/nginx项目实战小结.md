---
title: nginx项目使用总结
date: 2019-07-03 09:48:09
tags: nginx
---

### nginx

#### nginx cookie丢失问题  

系统通过域名解析经过阿里云的SLB解析进入内网通过nginx的转发到后端各服务，因为后端服务是标准的oauth2.0,请求过程中需要验证是否存在cookie，没有登录则重定向到登录页面，通过nginx的proxy_pass代理到后端服务时，发现cookie丢失，每次登录完之后又循环重定向到登录页面。通过配置proxy_cookie_path属性，同时登录成功后刷新页面会导致404，需要配置*try_files $uri /index.html* ，（此属性的作用按顺序检查文件是否存在，返回第一个找到的文件或文件夹(结尾加斜线表示为文件夹)，如果所有的文件或文件夹都找不到，会进行一个内部重定向到最后一个参数）。
![nginx系统架构图](/images/2019-7/2019-07-03.jpg)
[参考文档](https://blog.csdn.net/we_shell/article/details/45153885)

#### nginx websocket代理

```
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection $connection_upgrade;
这里的关键部分在于HTTP请求中多了如下头部：
Upgrade: websocket
Connection: Upgrade
```
![nginx最终配置图](/images/2019-7/2019-07-03.1.jpg)
[参考文档1](https://www.xncoding.com/2018/03/12/fullstack/nginx-websocket.html)
[参考文档2](https://www.hi-linux.com/posts/42176.html)
