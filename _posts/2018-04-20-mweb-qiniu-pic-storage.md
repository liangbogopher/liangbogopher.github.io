---
layout: post
keywords: MacOS MWeb 七牛云
title: MWeb 添加七牛云图床
date: 2018-04-20
categories: Tools
tags:
    - MWeb
    - 七牛云
---

使用`Hexo`搭建的静态博客以来，一直使用`markdown`文件记录博客，工具推荐使用 `MWeb`，之前上传图片到七牛云都是人工手动处理，特别麻烦，今天发现`MWeb`可以配置七牛云图床，实现本地编辑`markdown`文件时，可以自动上传本地图片到七牛云远端。

<!-- more -->

### 1. 打开偏好设置

`偏好设置`--> `发布服务`

![add_qiniu_service](http://oe7n2xiy9.bkt.clouddn.com/add_qiniu_service.png)

### 2. 添加七牛云配置

![qiniu_service_config](http://oe7n2xiy9.bkt.clouddn.com/qiniu_service_config.png)

其中：
名称：可随意填写
API地址：点击`？`，选择你七牛云所在的区域的上传域名
![qiniu_storge_area](http://oe7n2xiy9.bkt.clouddn.com/qiniu_storge_area.png)
空间名称：和你七牛云存储空间名称一致即可
Access Key和 Secret Key: 在**个人面板**的**密钥管理**里面查看
图片URL前缀：七牛云分配给你存储空间的测试域名，记得加上`http://`，不然自动生成的`markdown`图片地址有问题。

配置好了后，点击**验证**，来确定是否配置正确。

### 3. 配置OK

直接将本地拖入文本编辑区域，就会自动上传图片，以及生成`markdown`代码。

完美！

