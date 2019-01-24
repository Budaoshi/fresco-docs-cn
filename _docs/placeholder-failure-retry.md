---
docid: placeholder-failure-retry
title: 占位图, 失败图和重试图
layout: docs
permalink: /docs/placeholder-failure-retry.html
---

在加载网络图片时，事情可能不会那么顺利。它或许会花费较长的时间，甚至要加载的图片根本就不可用。我们已经知道如何显示[进度条](progress-bars.html)。本节，我们来看另一个问题：在目标图片不可用时，`SimpleDraweeView` 如何显示。值得主要的是，这些都有不同的[缩放类型](scaletypes.html)，你可以自定义它们。

### 占位图

在你设置 URI 或者 controller 前，占位图就会显示，直到本次加载请求结束（成功抑或失败）。

#### XML

```xml
<com.facebook.drawee.view.SimpleDraweeView
  android:id="@+id/my_image_view"
  android:layout_width="20dp"
  android:layout_height="20dp"
  fresco:placeholderImage="@drawable/my_placeholder_drawable"
  />
```

#### 代码

```java
mSimpleDraweeView.getHierarchy().setPlaceholderImage(placeholderImage);
```

### 失败图

在请求错误后会显示失败图。这可能是网络相关的错误（404，超时），也可能是图片本身的问题（图片异常，格式不支持）。

#### XML

```xml
<com.facebook.drawee.view.SimpleDraweeView
  android:id="@+id/my_image_view"
  android:layout_width="20dp"
  android:layout_height="20dp"
  fresco:failureImage="@drawable/my_failure_drawable"
  />
```

#### 代码

```java
mSimpleDraweeView.getHierarchy().setFailureImage(failureImage);
```

### 重试图

重试图会替代错误图而被显示。当用户在其上点击时，会重试 4 次，若一直失败则显示失败图。为了让重试图有效，你需要在你的 controller 中设置其使能。也就是说你可以像这样来设置你的图片请求：

```java
mSimpleDraweeView.setController(
    Fresco.newDraweeControllerBuilder()
        .setTapToRetryEnabled(true)
        .setUri(uri)
        .build());
```

#### XML

```xml
<com.facebook.drawee.view.SimpleDraweeView
  android:id="@+id/my_image_view"
  android:layout_width="20dp"
  android:layout_height="20dp"
  fresco:failureImage="@drawable/my_failure_drawable"
  />
```

#### 代码

```java
simpleDraweeView.getHierarchy().setRetryImage(retryImage);
```

### 更多阅读

占位图、失败图和重试图都是 Drawee 分支。 它们之所以在本节中介绍，是因为它们是最常使用的分支。若要了解所有的分支以及它们的工作原理，请查看 [Drawee 分支](drawee-branches.html)。

### 示例

showcase 应用有个 [DraweeHierarchyFragment](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/drawee/DraweeHierarchyFragment.java)，展示了对占位图、失败图和重试图的用户。

![带有占位图、失败图和重试图的 Showcase 应用](/static/images/docs/01-placeholder-sample.png)
