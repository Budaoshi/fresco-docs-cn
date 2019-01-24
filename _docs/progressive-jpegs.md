---
docid: progressive-jpegs
title: 渐进式 JPEG 图
layout: docs
permalink: /docs/progressive-jpegs.html
---

Fresco 支持渐进式的网络JPEG图。

在开始加载之后，图会从模糊到清晰渐渐呈现。

在下载图片扫描时，它们会显示在你的控件上。用户将看到图片的清晰度从模糊逐渐变得清晰。

渐进式JPEG图仅仅支持网络图。本地图片会一次解码完成，所以没必要渐进式加载。你还需要知道的是，并不是所有的JPEG图片都是渐进式编码的，所以对于这类图片，不可能做到渐进式加载。

## 构建图片请求

目前，你必须在构建图片请求时显示的请求渐进式渲染：

```java
Uri uri;
ImageRequest request = ImageRequestBuilder.newBuilderWithSource(uri)
    .setProgressiveRenderingEnabled(true)
    .build();
DraweeController controller = 
Fresco.newDraweeControllerBuilder()
    .setImageRequest(request)
    .setOldController(mSimpleDraweeView.getController())
    .build();
mSimpleDraweeView.setController(controller);
```

我们希望在后续的版本中，在`setImageURI`方法中可以直接支持渐进式图片加载。

### 完整示例

查看 showcase 应用中的 `ImageFormatProgressiveJpegFragment` 了解完整示例：
[ImageFormatProgressiveJpegFragment.java](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/imageformat/pjpeg/ImageFormatProgressiveJpegFragment.java)

<video controls="" autoplay="">
  <source src="/static/videos/01-progressive-jpegs.mp4" type="video/mp4">
</video>