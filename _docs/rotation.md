---
docid: rotation
title: 图片旋转
layout: docs
permalink: /docs/rotation.html
---

你可以在图片请求中指定一个旋转角度来旋转图片，就像这样：

```java
final ImageRequest imageRequest = ImageRequestBuilder.newBuilderWithSource(uri)
    .setRotationOptions(RotationOptions.forceRotation(RotationOptions.ROTATE_90))
    .build();
mSimpleDraweeView.setController(
    Fresco.newDraweeControllerBuilder()
        .setImageRequest(imageRequest)
        .build());
```


### 自动旋转

许多设备会在 JPEG 文件的 metadata 中记录下照片的方向。如果你想图片呈现的方向和设备屏幕的方向一致，你可以简单地这样做到:

```java
final ImageRequest imageRequest = ImageRequestBuilder.newBuilderWithSource(uri)
    .setRotationOptions(RotationOptions.autoRotate())
    .build();
mSimpleDraweeView.setController(
    Fresco.newDraweeControllerBuilder()
        .setImageRequest(imageRequest)
        .build());
```

### 复合旋转（Combining rotations）

如果在你加载的图片的 EXIF 数据中有旋转信息，调用 `forceRotation` 方法会为图片 **添加** 默认的旋转值。例如，如果 EXIF 头中指定了旋转 90 度，并且你调用了 `forceRotation(ROTATE_90)`，那么原始图片将会被旋转 180 度。

### 示例

Fresco showcase 应用有个 [DraweeRotationFragment](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/drawee/DraweeRotationFragment.java) 类展示了多种旋转设置项。你可以使用[这里](https://github.com/recurser/exif-orientation-examples)的图片来作为演示用例。

![带有旋转示例的 Showcase 应用](/static/images/docs/01-rotation-sample.png)
