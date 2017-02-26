---
layout:     post
title:      "行吧云购"
subtitle:   "行吧云购项目的详细说明"
date:       2016-12-21
author:     "XuZheng"
# header-img: "post-bg-unix-linux.jpg"
tags:
    - 行吧云购
    - Android开发
    - 后端
---

> 这款App的核心功能是在线看车、在线订购养车服务、在线预付购车定金等功能。

## 项目简介
行吧云购是一款具有在线看车、在线订购养车服务、在线预付购车定金等功能的App。其中包含Android端、IOS端、和Web端以及后台管理系统。
## 难点

## 技术

  - Android客户端

      开发环境：Mac os、AndroidStudio
  - 后台管理系统
## 问题
  - 车辆详细配置时，有超过100项以上的参数需要配置。如果手动录入参数，是一个耗时耗力的工程。

    解决方案：在后端采用java的POI操作，让用户直接导入excel的参数列表。将读取到的excel数据序列化后，在存储到数据库中。
  - 在客户端页面显示的时候有大量的细节需要考虑，比如动画效果、页面
