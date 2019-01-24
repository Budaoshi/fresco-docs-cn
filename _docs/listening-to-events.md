---
docid: listening-to-events
title: 监听事件
layout: docs
permalink: /docs/listening-to-events.html
---

## 动机

Fresco 内置了 image pipeline 和 view controller 基础接口(instrumentation interfaces)。我们可以使用这些接口来记录性能和响应事件。

Fresco 有两个主要的基础接口：

- `RequestListener` 是 `ImagePipelineConfig` 中注册的一个全局监听器，它用于记录所有生产者-消费者链处理的请求。
- `ControllerListener` 可以添加到特定的 `DraweeView`，方便的对事件做出响应，比如“这个图片加载完成”。

### ControllerListener

相对于 `RequestListener` 是一个全局监听器，`ControllerListener` 适用于某个特定的 `DraweeView`。“图片加载失败”或者“图片加载完成”，响应于这样的事件是个良好的实践。同时，继承 `BaseControllerListener` 实现是个不错的选择。


一个简单的监听器可能像下面这样：

```java
public class MyControllerListener extends new BaseControllerListener<ImageInfo>() {

  @Override
  public void onFinalImageSet(String id, ImageInfo imageInfo, Animatable animatable) {
    Log.i("DraweeUpdate", "Image is fully loaded!");
  }

  @Override
  public void onIntermediateImageSet(String id, ImageInfo imageInfo, Animatable animatable) {
    Log.i("DraweeUpdate", "Image is partly loaded! (maybe it's a progressive JPEG?)");
    if (imageInfo != null) {
      int quality = imageInfo.getQualityInfo().getQuality();
      Log.i("DraweeUpdate", "Image quality (number scans) is: " + quality);
    }
  }

  @Override
  public void onFailure(String id, Throwable throwable) {
    Log.i("DraweeUpdate", "Image failed to load: " + throwable.getMessage());
  }
}
```

按照下面方法将其添加到你的 `DraweeController`：

```java
DraweeController controller = Fresco.newDraweeControllerBuilder()
    .setImageRequest(request)
    .setControllerListener(new MyControllerListener())
    .build();
mSimpleDraweeView.setController(controller);
```

### RequestListener

`RequestListener` 监听器依赖于大量的回调方法。最重要的是，你将注意到，他们都提供了一个唯一的 `requestId`，这样你就可以在多个环节中来追踪图片请求了。

由于存在大量回调方法，我们建议你继承 `BaseRequestListener` 类，并且只实现你需要的方法。像下面这样来注册你的监听器：

```java
final Set<RequestListener> listeners = new HashSet<>();
listeners.add(new MyRequestLoggingListener());

ImagePipelineConfig imagePipelineConfig = ImagePipelineConfig.newBuilder(this)
  .setRequestListeners(listeners)
  .build();

Fresco.initialize(this, imagePipelineConfig);
```

我们从 showcase 应用的一个图片请求中来跟踪请求日志，以便来讨论它们的含义。在你运行 showcase 应用是，你可以通过 `adb logcat` 来查看这些日志：

```java
RequestLoggingListener: time 2095589: onRequestSubmit: {requestId: 5, callerContext: null, isPrefetch: false}
```

当图片请求 `ImageRequest` 进入 image pipeline 时，`onRequestSubmit(...)` 会被调用。在这里你可以利用调用者上下文对象（caller context object）来识别出谁在发送请求：

```java
RequestLoggingListener: time 2095590: onProducerStart: {requestId: 5, producer: BitmapMemoryCacheGetProducer}
RequestLoggingListener: time 2095591: onProducerFinishWithSuccess: {requestId: 5, producer: BitmapMemoryCacheGetProducer, elapsedTime: 1 ms, extraMap: {cached_value_found=false}}
```

所有的生产者都调用了 `onProducerStart(...)` 和 `onProducerFinishWithSuccess(...)` (或者 `onProducerFinishWithFailure(...)`) 方法。 上面的日志是在检查 Bitamp 缓存。

```java
RequestLoggingListener: time 2095592: onProducerStart: {requestId: 5, producer: BackgroundThreadHandoffProducer}
RequestLoggingListener: time 2095593: onProducerFinishWithSuccess: {requestId: 5, producer: BackgroundThreadHandoffProducer, elapsedTime: 1 ms, extraMap: null}
RequestLoggingListener: time 2095594: onProducerStart: {requestId: 5, producer: BitmapMemoryCacheProducer}
RequestLoggingListener: time 2095594: onProducerFinishWithSuccess: {requestId: 5, producer: BitmapMemoryCacheProducer, elapsedTime: 0 ms, extraMap: {cached_value_found=false}}
RequestLoggingListener: time 2095595: onProducerStart: {requestId: 5, producer: EncodedMemoryCacheProducer}
RequestLoggingListener: time 2095596: onProducerFinishWithSuccess: {requestId: 5, producer: EncodedMemoryCacheProducer, elapsedTime: 1 ms, extraMap: {cached_value_found=false}}
RequestLoggingListener: time 2095596: onProducerStart: {requestId: 5, producer: DiskCacheProducer}
RequestLoggingListener: time 2095598: onProducerFinishWithSuccess: {requestId: 5, producer: DiskCacheProducer, elapsedTime: 2 ms, extraMap: {cached_value_found=false}}
RequestLoggingListener: time 2095598: onProducerStart: {requestId: 5, producer: PartialDiskCacheProducer}
RequestLoggingListener: time 2095602: onProducerFinishWithSuccess: {requestId: 5, producer: PartialDiskCacheProducer, elapsedTime: 4 ms, extraMap: {cached_value_found=false}}
```

在请求交由后台线程(`BackgroundThreadHandoffProducer`)处理时，我们可以看到更多的日志信息

```java
RequestLoggingListener: time 2095602: onProducerStart: {requestId: 5, producer: NetworkFetchProducer}
RequestLoggingListener: time 2095745: onProducerEvent: {requestId: 5, stage: NetworkFetchProducer, eventName: intermediate_result; elapsedTime: 143 ms}
RequestLoggingListener: time 2095764: onProducerFinishWithSuccess: {requestId: 5, producer: NetworkFetchProducer, elapsedTime: 162 ms, extraMap: {queue_time=140, total_time=161, image_size=40502, fetch_time=21}}
RequestLoggingListener: time 2095764: onUltimateProducerReached: {requestId: 5, producer: NetworkFetchProducer, elapsedTime: -1 ms, success: true}
```

在这部分请求中，`NetworkFetchProducer` 是 “终极生成者”。也就是说，它为完成本次请求提供了确切的输入源。如果图片被已缓存，`DiskCacheProducer` 将会是“终极”生成者。

```java
RequestLoggingListener: time 2095766: onProducerStart: {requestId: 5, producer: DecodeProducer}
RequestLoggingListener: time 2095786: onProducerFinishWithSuccess: {requestId: 5, producer: DecodeProducer, elapsedTime: 20 ms, extraMap: {imageFormat=JPEG, ,hasGoodQuality=true, bitmapSize=788x525, isFinal=true, requestedImageSize=unknown, encodedImageSize=788x525, sampleSize=1, queueTime=0}
RequestLoggingListener: time 2095788: onRequestSuccess: {requestId: 5, elapsedTime: 198 ms}
```
同样的，`DecodeProducer` 成功执行，`onRequestSuccess(...) 最终被调用。

你会注意到这些方法中的大多数都提供了一个可选信息放入 `Map<String, String> extraMap`。那些用于遍历元素的字符串常量通常是对应的生成者类中的公有常量。