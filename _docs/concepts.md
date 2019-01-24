---
docid: concepts
title: 关键概念
layout: docs
permalink: /docs/concepts.html
---

## Drawees

Drawees 负责图片的呈现。它由三个元素组成，有点像MVC模式。

### DraweeView

继承自 [View](http://developer.android.com/reference/android/view/View.html), 负责图片的显示。

一般情况下，使用 `SimpleDraweeView` 即可。 你可以在 XML 或者在 Java 代码中使用它，通过 `setImageUri` 给它设置一个 URI 来使用，这里有简单的入门教学：[入门指南](index.html) 

查看 [使用 SimpleDraweeView](using-simpledraweeview.html) 一节。

### DraweeHierarchy

DraweeHierarchy 用于组织和维护最终绘制和呈现的 [Drawable](http://developer.android.com/reference/android/graphics/drawable/Drawable.html) 对象，相当于MVC中的M。

查看 [使用 SimpleDraweeView](using-simpledraweeview.html) 一节。

### DraweeController

`DraweeController` 负责和 image loader 交互（ Fresco 中默认为 image pipeline, 当然你也可以指定别的），可以创建一个这个类的实例，来实现对所要显示的图片做更多的控制。

如果你还需要对Uri加载到的图片做一些额外的处理，那么你会需要这个类的。

### DraweeControllerBuilder

`DraweeControllers` 由构建者模式[构建](using-controllerbuilder.html)，创建之后，不可修改。

### Listeners

使用 `ControllerListener` 的一个场景就是设置一个 [Listener监听图片的下载](listening-to-events.html)。

## The Image Pipeline

Fresco 的 Image Pipeline 负责图片的获取和管理。图片可以来自远程服务器，本地文件，或者Content Provider，本地资源。压缩后的文件缓存在本地存储中，Bitmap数据缓存在内存中。

在5.0系统以下，Image Pipeline 使用 *pinned purgeables* 将Bitmap数据避开Java堆内存，存在ashmem中。这要求图片不使用时，要显式地释放内存。

`SimpleDraweeView`自动处理了这个释放过程，所以没有特殊情况，尽量使用SimpleDraweeView，在特殊的场合，如果有需要，也可以直接控制Image Pipeline。

