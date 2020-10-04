---
title: fisco-browser项目架构
date: 2019-10-22 10:08:41
tags: java
---
<details>
  <summary>详细信息</summary>
从微众的fisco-browser开源区块链浏览器中学习java和后端项目架构。

目录结构：

- base
- config 
- controller
- entity
- mapper
- schedule
- service
- util



base: 基础，包括错误处理exception文件夹，所有controller的父类BaseController,自定义状态码ConstantCode，String类型常量Constants，状态码对象RetCode(包含一个String 和一个Integer类型)



config: 基础配置，具体还没太弄明白，以后填坑



controller：路由



entity: 数据实体定义，由service层调用，返回给前端



mapper： 数据到数据库的映射



schedule:  用springboot自带的定时任务功能将区块链上数据导入数据库



service： 由路由层调用，建立实体对象，从数据库中拿数据给实体对象，处理逻辑，返回



util: 组件，包括Web3Rpc区块链交互组件,读取压缩文件,时间组件等等，暂时没弄太明白，等有时间填坑

</details>