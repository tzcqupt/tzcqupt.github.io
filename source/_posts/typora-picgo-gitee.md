---
title: Typora使用PicGo上传图片到Gitee
Author: tzcqupt
categories: [软件]
tags: [软件安装]
comments: true
toc: true
date: 2020-05-04 00:00:00
---

# 环境准备

## Node.js

进入 nodejs 官网：https://nodejs.org/en/download/，直接下载安装对应的版本即可。

> 版本>=8

## Typora

进入Typora官网下载 https://www.typora.io/#windows

> 软件版本>=0.9.86

## PicGo

进入 PicGo 官网：https://github.com/Molunerfinn/PicGo/releases ，直接下载安装。

## 码云仓库

进入码云官网https://gitee.com/ 新建公有仓库，专门来存放图片。

# 软件配置

## 插件下载

1. 进入 PicGo，选择 插件设置，输入 gitee，选择安装第二个插件 **gitee-uploader 1.1.2 **，该插件用来设置 gitee 作为图床的。

2. 安装成功后，点击该插件右下角齿轮状图标（设置），选择 uploader - gitee

   > 可设置软件开机启动

## 插件配置

其中 repo 为 : giteeUsername/repositoryName，比如你的 gitee 账号为 user，存储图片的仓库为 blog-image，则此处填入 blog-image

token 则是 gitee 上个人的私人令牌，直接填入（私人令牌如何生成请看下一步说明）

然后点击 确定

## 生成Token私人令牌

1. 私人令牌生成的话，首先进入到 个人的设置 中，在左侧有个 私人令牌，点击进入
2. 然后可以看到右侧有个 +生成新令牌 ，点击
3. 填写 私人令牌描述，比如：PicGo，这个仅仅用来标记，关系不大，点击 提交
4. 输入账号密码，点击 验证
5. 最后，成功生成私人令牌，点击 复制 放到上一步中的 token 中

# Typora使用PicGo

1. 打开偏好设置，设置picGo的安装路径

![](https://gitee.com/tzcqupt/blog-image/raw/master/img/typora设置picGo.PNG)

2. 插入图片，可以使用`Ctrl+Shift+I`插入，或者输入`![]()`来选择本地的图片，这样会进行自动上传到gitee。

# 参考文章

[利用码云 gitee + PicGo 搭建个人免费图床](https://msd.misuland.com/pd/4146263467944314736)

