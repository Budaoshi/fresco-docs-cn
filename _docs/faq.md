---
docid: faq
title: 常见问题解答
layout: docs
permalink: /docs/faq.html
---

这里是我们的 Github 上经常被问到的问题。如果你有任何的疑问欢迎给我们提交 pull-request。

### 我该如何清除所有缓存？

你可以使用如下代码来删除缓存中的所有图片（包括磁盘缓存和内存缓存）：

```java
// 清除内存和磁盘缓存
Fresco.getImagePipeline().clearCaches();
```

### 我如何创建一个支持缩放手势的 Drawee？

在我们的 Sample 中，你可以找到ZoomableDraweeView](https://github.com/facebook/fresco/tree/master/samples/zoomable)模块来查看相关代码。

### 我该怎样为本地文件生成一个 URI？

使用 `UriUtil` 工具类:

```java
final File file = new File("your/file/path/img.jpg");
final URI uri = UriUtil.getUriForFile(file);
```

### 我该怎样为资源文件生成 URI？

使用 `UriUtil` 工具类:

```java
final int resourceId = R.drawable.my_image;
final URI uri = UriUtil.getUriForResourceId(resourceId);

// alternatively, if it is from another package:
final URI uri = UriUtil.getUriForQualifiedResource("com.myapp.plugin", resourceId);
```

### 在 RecyclerView 中如何使用 Fresco ？

像使用其它任何 `RecyclerView` 一样创建你的 `RecyclerView`。`DraweeView` 能够自动完成 attach 和 detach。当被 detach 时它会自动清除图片内存。当重新被 attach 时，如果 BitmapCache 存在则会被优先加载。

更多内容查看我们 showcase app 中 [DraweeRecyclerViewFragment.java](https://github.com/facebook/fresco/blob/1472a3e1b1655e9b52c74e0b06d5ba60d15a42f9/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/drawee/DraweeRecyclerViewFragment.java) 部分。

### 如何下载未解码的图片？

你可以使用 image pipeline 中的 `imagePipeline#fetchEncodedImage(ImageRequest, ...)` 方法来达到此目的。在 [直接使用Image Pipeline](using-image-pipeline.html) [数据源和数据订阅者](datasources-datasubscribers.html) 小节有翔实的示例。

### 在展示前我该如何修改图片？

最好的方式是实现 [后处理器](post-processor.html)。它允许 image pipeline 可以在后台完成更改并高效地分配 Bitmaps。

### Fresco 有多大？

如果按照 [应用中引入 Fresco](proguard.html)一节中的步骤正确操作，你的发布版本不会比之前多出 500KB。

在老设备上添加对动画(`com.facebook.fresco:animated-gif`, `com.facebook.fresco:animated-webp`)和 WebP(`com.facebook.fresco:webpsupport`) 的支持是可选的。

模块化的方式可以让 Fresco 的基础库更加轻量化。每添加一个可选库会增加大概 100KB 大小。

### DraweeView 为什么不能使用 Android 提供的 wrap_content 属性？

一方面是因为 Drawee 的 [getIntrinsicHeight](http://developer.android.com/reference/android/graphics/drawable/Drawable.html#getIntrinsicHeight()) 和 getIntrinsicWidth 方法总是返回 -1.

另一方面是因为与简单的 ImageView 控件不同，Drawee 可能同一时间会显示两张图片。比方说，在占位图到目标图渐变过程中，Drawee 需要让两者都可见。甚至会有多张目标图片需要展示，一张是低分辨率图片，另一张是高清图。如果这些图片大小不一致，那么 “intrinsic” 大小这一概念就不容易定义。

在目标图片未加载完成前，我们可以返回占位图的大小，待完成后切换到目标图片的大小。如果我们这样做的话，图片将不能正常展示----它可能会被缩放或裁剪成占位图片的大小。唯一能够避免的方法就是在图片加载后强制让 Android 系统重新布局。这样不但破坏了应用的滑动偏好，也会让你的用户感到不适，他们会看到你的应用突然的发生变化。设想一下，你的用户正在阅读一篇文章，突然文字跳到了底部，这些都是由于图片刚刚加载完成触发系统重新布局的缘故。
由于上述原因，你需要使用确定值或者 `match_parent` 来放置你的 DraweeView。

如果你的图片来自服务器，你可以在下载图片前向服务器询问该图的尺寸。这不是一个耗时请求。然后使用 [setLayoutParams](http://developer.android.com/reference/android/view/View.html#setLayoutParams(android.view.ViewGroup.LayoutParams)) 方法来提前动态改变视图大小。

另一方面，如果你确实有这样的需求，你可以按照[这里](http://stackoverflow.com/a/34075281/3027862)的描述，使用 controller 监听器来动态改变 Drawee 视图的大小。请注意，我们有意识的移除了这个功能，因为这不是我们期望的。[丑陋的事情看起来就是丑陋的](https://youtu.be/qCdpTji8nxo?t=890)。
