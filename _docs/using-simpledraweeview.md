---
docid: using-simpledraweeview
title: 使用 SimpleDraweeVIew
layout: docs
permalink: /docs/using-simpledraweeview.html
---

在你使用 Fresco 时，你会用到 `SimpleDraweeView` 来显示图片。可以在 XML 布局中使用。最简单的使用方式如下：

```xml
<com.facebook.drawee.view.SimpleDraweeView
  android:id="@+id/my_image_view"
  android:layout_width="20dp"
  android:layout_height="20dp"
  />
```

**注意：** `SimpleDraweeView` 不支持 `layout_width` 或 `layout_height` 属性的 `wrap_content`。更多信息可以查看[这里](faq.html)。唯一的特例是你指定了宽高比，像这样：

```xml
<com.facebook.drawee.view.SimpleDraweeView
    android:id="@+id/my_image_view"
    android:layout_width="20dp"
    android:layout_height="wrap_content"
    fresco:viewAspectRatio="1.33"
    />
```

### 加载图片

最简单的加载图片到 `SimpleDraweeView` 控件中的方式是调用 `setImageURI` 方法：

```java
mSimpleDraweeView.setImageURI(uri);
```

就这样简单，你现在已经使用 Fresco 可以显示图片了！

### 高阶 XML 属性

`SimpleDraweeView`，“名不副实“，支持许多自定义的属性。下面的例子展示了它所有的属性信息：

```xml
<com.facebook.drawee.view.SimpleDraweeView
  android:id="@+id/my_image_view"
  android:layout_width="20dp"
  android:layout_height="20dp"
  fresco:fadeDuration="300"
  fresco:actualImageScaleType="focusCrop"
  fresco:placeholderImage="@color/wait_color"
  fresco:placeholderImageScaleType="fitCenter"
  fresco:failureImage="@drawable/error"
  fresco:failureImageScaleType="centerInside"
  fresco:retryImage="@drawable/retrying"
  fresco:retryImageScaleType="centerCrop"
  fresco:progressBarImage="@drawable/progress_bar"
  fresco:progressBarImageScaleType="centerInside"
  fresco:progressBarAutoRotateInterval="1000"
  fresco:backgroundImage="@color/blue"
  fresco:overlayImage="@drawable/watermark"
  fresco:pressedStateOverlayImage="@color/red"
  fresco:roundAsCircle="false"
  fresco:roundedCornerRadius="1dp"
  fresco:roundTopLeft="true"
  fresco:roundTopRight="false"
  fresco:roundBottomLeft="false"
  fresco:roundBottomRight="true"
  fresco:roundTopStart="false"
  fresco:roundTopEnd="false"
  fresco:roundBottomStart="false"
  fresco:roundBottomEnd="false"
  fresco:roundWithOverlayColor="@color/corner_color"
  fresco:roundingBorderWidth="2dp"
  fresco:roundingBorderColor="@color/border_color"
  />
```

### 代码中自定义

尽管我们推荐你在 XML 文件中设置这些属性，但他们同样可以在代码中设置。为此，你需要在设置图片 URI 前创建一个 `DraweeHierarchy` 对象：

```java
GenericDraweeHierarchy hierarchy =
    GenericDraweeHierarchyBuilder.newInstance(getResources())
        .setActualImageColorFilter(colorFilter)
        .setActualImageFocusPoint(focusPoint)
        .setActualImageScaleType(scaleType)
        .setBackground(background)
        .setDesiredAspectRatio(desiredAspectRatio)
        .setFadeDuration(fadeDuration)
        .setFailureImage(failureImage)
        .setFailureImageScaleType(scaleType)
        .setOverlays(overlays)
        .setPlaceholderImage(placeholderImage)
        .setPlaceholderImageScaleType(scaleType)
        .setPressedStateOverlay(overlay)
        .setProgressBarImage(progressBarImage)
        .setProgressBarImageScaleType(scaleType)
        .setRetryImage(retryImage)
        .setRetryImageScaleType(scaleType)
        .setRoundingParams(roundingParams)
        .build();
mSimpleDraweeView.setHierarchy(hierarchy);
mSimpleDraweeView.setImageURI(uri);
```

**注意：** 它们中的有些属性可以在已存在的 hierarchy 中设置而不用创建新的。为此，我们可以简单的通过 `SimpleDraweeView` 获取一个 hierarchy，并调用它的方法设置，比如：

```java
mSimpleDraweeView.getHierarchy().setPlaceHolderImage(placeholderImage);
```

### 完整示例

可以在 showcase 应用的 `DraweeSimpleFragment` 中查看完整示例：[DraweeSimpleFragment.java](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/drawee/DraweeSimpleFragment.java)

![带有缩放类型的 Showcase 应用](/static/images/docs/01-using-simpledraweeview-sample.png)
