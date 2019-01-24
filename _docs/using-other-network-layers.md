---
docid: using-other-network-layers
title: 自定义网络加载
layout: docs
permalink: /docs/using-other-network-layers.html
---

Image pipeline 默认使用[HttpURLConnection](https://developer.android.com/training/basics/network-ops/connecting.html)。应用可以根据自己需求使用不同的网络库。Fresco 已经包含了一个可替代的网络加载库，它基于 OkHttp 实现。

### 使用 OkHttp

[OkHttp](http://square.github.io/okhttp) 是一个流行的开源网络请求库。

### 1. 设置 Gradle

若要使用它，你需要对你的 `build.gradle` 文件中的 `dependencies` 部分做修改。顺着[入门指南](index.html) 部分给出的 Gradle 依赖，你只需要添加一行：

对于 OkHtt2:

```groovy
dependencies {
  // 工程中的其它依赖
  compile "com.facebook.fresco:imagepipeline-okhttp:{{site.current_version}}+"
}
```

对于 OkHttp3:

```groovy
dependencies {
  // 工程中的其它依赖
  compile "com.facebook.fresco:imagepipeline-okhttp3:{{site.current_version}}+"
}
```

#### 配置 image pipeline 使用 OkHttp

配置Image
pipeline这时也有一些不同，不再使用`ImagePipelineConfig.newBuilder`,而是使用`OkHttpImagePipelineConfigFactory`:

```java
Context context;
OkHttpClient okHttpClient; // build on your own
ImagePipelineConfig config = OkHttpImagePipelineConfigFactory
    .newBuilder(context, okHttpClient)
    . // other setters
    . // setNetworkFetcher is already called for you
    .build();
Fresco.initialize(context, config);
```

更多有关它的详细示例，查看在[Fresco showcase 应用](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/ShowcaseApplication.java)中如何配置使用。

### 处理 Session 和 Cookie

你传给`OkHttpClient`需要处理服务器的安全校验工作（可以通过Interceptor处理）。参考[这个bug](https://github.com/facebook/fresco/issues/385) 来处理自定义网络库可能发生的 Cookie 相关的问题。

### 使用自定的网络层（可选）

为了完全控制网络层的行为，你可以自定义网络层。继承[NetworkFetchProducer](../javadoc/reference/com/facebook/imagepipeline/producers/NetworkFetchProducer.html), 这个类包含了网络通信。你也可以选择性地继承[FetchState](../javadoc/reference/com/facebook/imagepipeline/producers/FetchState.html), 这个类是请求时的数据结构描述。

默认的 `OkHttp 3` 的实现可以作为一个参考. [源码在这](https://github.com/facebook/fresco/blob/master/imagepipeline-backends/imagepipeline-okhttp3/src/main/java/com/facebook/imagepipeline/backends/okhttp3/OkHttpNetworkFetcher.java)。

在[配置Image pipeline](configuring-image-pipeline.html)时，把网络 producer 传递给 Image pipeline。

```java
ImagePipelineConfig config = ImagePipelineConfig.newBuilder()
  .setNetworkFetcher(myNetworkFetcher);
  . // other setters
  .build();
Fresco.initialize(context, config);
```

你可以像使用其它 URI 一样来加载动态的 WebP 图片。为了能自动播放动画，你可以在 `DraweeController` 中设置 `setAutoPlayAnimations(true)`：

```java
DraweeController controller = Fresco.newDraweeControllerBuilder()
    .setUri("http://example.org/somefolder/animated.webp")
    .setAutoPlayAnimations(true)
    .build();
mSimpleDraweeView.setController(controller);
```

### 完整示例

在 Showcase 应用的 `ImageFormatWebpFragment` 中查看完整示例：[ImageFormatWebpFragment.java](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/imageformat/webp/ImageFormatWebpFragment.java)

![带有通知的 Showcase 应用](/static/images/docs/03-webp-support-sample.png)