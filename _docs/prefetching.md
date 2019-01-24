---
docid: prefetching
title: 图片的预加载
layout: docs
permalink: /docs/prefetching.html
---

预加载图片提前对它们进行展示有时候可以缩短用户的等待时间。但是，请记住，这样做是需要代价的。预加载会占用用户的数据空间，共享用户的 CPU 和内存。因此，对于大多数应用，预加载是不推荐的。

尽管如此，image pipeline 允许你将图片预加载到磁盘或者 bitmap 缓存。对于网络图片，它们都会使用更多的数据，但是磁盘缓存不会对图片解码，因此会使用较少的 CPU。

__注意:__ 需要意识到，如果你的网络获取器(network fetcher)不支持优先级，对于那些需要立即显示到屏幕的图片，预加载可能会拖慢它们的加载速度。`OkHttpNetworkFetcher` 和 `HttpUrlConnectionNetworkFetcher` 当前都不支持优先级。

预加载至磁盘：

```java
imagePipeline.prefetchToDiskCache(imageRequest, callerContext);
```

预加载至 bitmap 缓存：

```java
imagePipeline.prefetchToBitmapCache(imageRequest, callerContext);
```

取消预加载：

```java
// keep the reference to the returned data source.
DataSource<Void> prefetchDataSource = imagePipeline.prefetchTo...;

// later on, if/when you need to cancel the prefetch:
prefetchDataSource.close();
```

在预加载完成后关闭预加载数据源是一个空操作，关闭它是绝对安全的。

### 示例

查看我们的 [showcase 应用](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/imagepipeline/ImagePipelinePrefetchFragment.java) 来了解如何使用预加载功能的一个示例。
