---
title: React Native 构建的仿豆瓣应用
date: '2020-05-24'
tags: ['ReactNative']
summary: 2020年写的一个 React Native 应用，简单但完整的练手 demo。包含常见的 React Native 库，网络，状态管理，主题...麻雀虽小五脏俱全
---

介绍一个用 React Native 创建的应用，集合了豆瓣电影，图书等信息展示功能的 app。（React Native 学习 demo 应用）

github 地址 → [【瓣芽·Banya】](https://github.com/lexguy/Banya_ReactNative)

项目使用了 react-navigation 做路由。redux 做部分状态管理，redux-persist 做数据本地化 。采用了少量的第三方库，并同时兼容 Android 和 iOS 端。项目适合初级 RN 开发者，对于 React Native 的学习具有一定的借鉴意义。

#### 1. 功能

- 电影榜单以及电影，影人详情信息展示
- 本地收藏电影
- 根据定位获取实时上映数据
- 预告片播放，图片缩放查看
- 数据存储，部分数据支持本地获取
- 主题换肤，全局设置自定义颜色

#### 2. 基本配置：

- react-navigation 　路由管理
- react-redux 　状态管理
- redux-persist 　数据持久化
- prop-types 　类型检查

#### 3. 主要第三方库：

- react-native-swiper 　轮播组件
- react-native-modal 　弹出框组件
- react-native-linear-gradient 　色彩渐变组件
- react-native-image-zoom-viewer 　图片预览查看组件
- react-native-video 　视频播放组件
- react-native-scrollable-tab-view 　页面切换 tab-view 组件

#### 4. 运行截图

<img src="https://pic.downk.cc/item/5eca95aac2a9a83be549daa9.png" />

换肤蓝色之后：

<img src="https://pic.downk.cc/item/5eca95aac2a9a83be549daaf.png" />

#### 5 .项目结构

- component 　自定义组件
- constant 　配置常量，资源文件
- navigation 　路由配置
- redux 　 redux 状态数据管理
- util 　 网络请求等公共函数
- views 　 screen 页面

<img src="https://ae01.alicdn.com/kf/H3fd218b8e1a546928ccf3ff86116ae91l.png"  width="300"  height="600" />

以上是个人开源项目，任何问题欢迎交流，共同学习！

github 地址 → [【瓣芽·Banya】](https://github.com/lexguy/Banya_ReactNative)
