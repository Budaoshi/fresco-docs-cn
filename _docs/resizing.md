---
docid: resizing
title: Resizing
layout: docs
permalink: /docs/resizing.html
---

本节中我们会用到如下术语：

**Scaling** 是一种画布操作，通常是由硬件加速的。图片实际大小保持不变，它只不过在绘制时被放大或缩小。参看[缩放类型](scaletypes.html)

**Resizing** 是一种软件执行的管道操作。它会在图片解码前改变内存中的编码图片。解码后的图片将比原图更小。

**Downsampling** 同样是软件实现的管道操作。它不是创建一张新的图片，而是只解码部分像素，以此来减少输出 bitmap 的大小。

### Resizing

Resize 并不改变原始图片，它只在解码前修改内存中的图片大小。

如果要 resize，创建`ImageRequest`时，提供一个 `ResizeOptions` :

```java
ImageRequest request = ImageRequestBuilder.newBuilderWithSource(uri)
    .setResizeOptions(new ResizeOptions(50, 50))
    .build();
mSimpleDraweeView.setController(
    Fresco.newDraweeControllerBuilder()
        .setOldController(mSimpleDraweeView.getController())
        .setImageRequest(request)
        .build());
```
Resize 有以下几个限制：

- 只支持 JPEG 文件
- 真实缩减的大小接近原图的 1/8
- 它不能让你的图片变得更大，只会更小

### Downsampling

Downsampling 是一个正在实验中的特性。使用的话需要在设置 image pipeline 时进行[设置](configure-image-pipeline.html)：

```java
   .setDownsampleEnabled(true)
```

如果开启该选项，pipeline 会向下采样（downsample）你的图片，代替 resize 操作。你仍然需要像上面那样在每个图片请求中调用 `setResizeOptions` 。

Downsampling 在大部分情况下比 resize 更快，因为它是解码的一部分，而不是独立部分。除了支持 JPEG 图片，它还支持 PNG 和 WebP(除动画外) 图片。

但是目前还有一个问题是它在 Android 4.4 上会在解码时造成更多的内存开销（相比于Resizing）。这在同时解码许多大图时会非常显著，我们希望在将来的版本中能够解决它并默认开启此选项。

## 应该使用哪种 ？

如果图片不是比 `View` 大出很多的情况下，只需要做 **Scaling**。 它更快，更容易书写代码，并且输出的图像质量更好。当然，图片比 `View`小的情况也是这样，这样你放大图片时，不会因为产生一张新的大图而浪费内存空间，却又没有带来更好的视觉提高。

“大出很多”的定义是：

> 图片的像素数 > 视图量级x 2

* 视图量级 = 长 x 宽 *

这在大部分的照相机拍摄图片下是符合的。假设你的设备屏幕尺寸是1080 x 1920 （大概2MP），那么一个16MP的照相机产生的相片比屏幕尺寸量级大8倍。此时毫无疑问要选择 **Resize**。

对于网络图片，在考虑 resize 之前，先尝试请求大小合适的图片。如果服务器能返回一张较小的图，就不要请求一张高解析度图片。你应该考虑用户的流量。同时，获取较小的图片可以减少你的 APP 的存储空间和 CPU 占用。

### 示例

Fresco showcase 应用有个 [ImagePipelineResizingFragment](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/imagepipeline/ImagePipelineResizingFragment.java) 类诠释了如何使用占位图，失败图和重试图。

![带有 resizing 示例的 Showcase 应用](/static/images/docs/01-resizing-sample.png)


