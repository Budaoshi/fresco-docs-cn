---
docid: sample-code
title: 示例代码
layout: docs
permalink: /docs/sample-code.html
---

## 译者DEMO

官方的项目，编译起来比较困难，如果你仅仅是想看 DEMO 运行效果，我将 DEMO 抽离出来，你直接使用这个[github项目](https://github.com/liaohuqiu/fresco-demo-for-gradle)

*以下原文*

*Note: 示例代码在“非商业使用” License 下，而不是 Fresco 的 MIT License。

你可以在Fresco的Github页面找到一些示例代码，他们只能配合源码一起运行。你需要参考[构建源码](building-from-source.html)。 

### Showcase app

[Showcase App](https://github.com/facebook/fresco/blob/master/samples/showcase) 展示了学多特性并允许你自定义参数来显示效果。
它包括了 Drawee 和 image pipeline 相关的代码。此外，它还展示了如何使用内置的和自定义的图片格式。

### 缩放库

[zoomable library](https://github.com/facebook/fresco/blob/master/samples/zoomable) 实现了一个`ZoomableDraweeView`支持手势缩放。

### 图片库比较 App

在这个App中我们比较了[Picasso](http://square.github.io/picasso), [Universal Image Loader](https://github.com/nostra13/Android-Universal-Image-Loader), [Volley](https://developer.android.com/training/volley/index.html), [Glide](https://github.com/bumptech/glide)。

你也可以比较Fresco中使用 OkHttp 和 Volley 的性能差异。

你可以设置图片源于网络或者本地相册，网络图片来源于[Imgur](http://imgur.com).

你可以运行自动化对比测试脚本[run_comparison.py](https://github.com/facebook/fresco/blob/master/run_comparison.py)。 下面这个指令会在一个 ARM-v7 设备上运行脚本:

```./run_comparison.py -c armeabi-v7a```

### 圆角示例 

示例了各类圆角、圆形图的用法

