---
docid: caching
title: 缓存
layout: docs
permalink: /docs/caching.html
prev: configure-image-pipeline.html
next: using-image-pipeline.html
---


Fresco 以三种不同的缓存方式来存储图片，它们分层管理，但是会增加你遍历图片时的开销。

#### 1. Bitmap缓存

Bitmap缓存存储`Bitmap`对象，这些Bitmap对象可以立刻用来显示或者用于[后处理](post-processor.html)

在5.0以下系统，Bitmap缓存位于 *ashmem*，这样Bitmap对象的创建和释放将不会引发GC，更少的GC会使你的APP运行得更加流畅。

相比之下，5.0及其以上系统，内存管理有了很大改进，所以Bitmap缓存直接位于Java的heap上。

当应用在后台运行时，该内存会被[清空](#clearing-the-cache)。

#### 2. 未解码图片的内存缓存

这个缓存存储的是原始压缩格式的图片。从该缓存取到的图片在使用之前，需要先进行解码。

如果有[resizing](resizing.html)、[旋转](rotation.html)，或者[WebP编码转换](#webp)工作需要完成，这些工作会在解码之前进行。

#### 3. 磁盘缓存

（是的，我们知道手机没有磁盘，但是 *本地存储缓存* 术语过于冗长...）

和未解码的内存缓存相似，磁盘缓存存储的是未解码的原始压缩格式的图片，在使用之前同样需要经过解码等处理。

和其它缓存形式不一样，APP在后台时，内容是不会被清空的。即使关机也不会。

当磁盘缓存达到 [DiskCacheConfig](configure-image-pipeline.html#configuring-the-disk-cache) 设置的大小限制时，Fresco 使用 LRU 算法来清理该部分缓存(查看 [DefaultEntryEvictionComparatorSupplier.java](https://github.com/facebook/fresco/blob/master/imagepipeline-base/src/main/java/com/facebook/cache/disk/DefaultEntryEvictionComparatorSupplier.java))。

当然，用户可以随时从系统的设置菜单中进行清空缓存操作。

#### 查找一个bitmap是否被缓存？

你可以使用[ImagePipeline](../javadoc/reference/com/facebook/imagepipeline/core/ImagePipeline.html)中提供的方法检查bitmap是否在缓存中。

```java
ImagePipeline imagePipeline = Fresco.getImagePipeline();
Uri uri;
boolean inMemoryCache = imagePipeline.isInBitmapMemoryCache(uri);
```

由于磁盘检测必须运行在另一个线程中，磁盘缓存的检测是异步执行的。你可以像这样使用：

```java
DataSource<Boolean> inDiskCacheSource = imagePipeline.isInDiskCache(uri);
DataSubscriber<Boolean> subscriber = new BaseDataSubscriber<Boolean>() {
    @Override
    protected void onNewResultImpl(DataSource<Boolean> dataSource) {
      if (!dataSource.isFinished()) {
        return;
      }
      boolean isInCache = dataSource.getResult();
      // 这里是你的代码
    }
  };
inDiskCacheSource.subscribe(subscriber, executor);
```

以上API假设你使用默认的CacheKeyFactory。如果你自定义`CacheKeyFactory`，你可能需要用把ImageRequest作为它的参数。

### 清除缓存中的一条url

[ImagePipeline](../javadoc/reference/com/facebook/imagepipeline/core/ImagePipeline.html)现有函数可以根据Uri删除缓存。

```java
ImagePipeline imagePipeline = Fresco.getImagePipeline();
Uri uri;
imagePipeline.evictFromMemoryCache(uri);
imagePipeline.evictFromDiskCache(uri);

// 以上两中方法的综合式
imagePipeline.evictFromCache(uri);
```

如同上面一样，`evictFromDiskCache(Uri)`假定你使用的是默认的CacheKeyFactory。如果你自定义，请使用`evictFromDiskCache(ImageRequest)`。

### 清除缓存

```java
ImagePipeline imagePipeline = Fresco.getImagePipeline();
imagePipeline.clearMemoryCaches();
imagePipeline.clearDiskCaches();

// 以上两中方法的综合式
imagePipeline.clearCaches();
```

### 用一个还是两个磁盘缓存?

大部分的应用有一个磁盘缓存就够了，但是在一些情况下，你可能需要两个缓存。比如你也许想把小文件放在一个缓存中，大文件放在另外一个文件中，这样小文件就不会因大文件的频繁变动而被从缓存中移除。

为此，我们只需要在[配置 image pipeline](configure-image-pipeline.html)时调用 `setMainDiskCacheConfig` 或者 `setSmallImageDiskCacheConfig` 方法即可。

至于什么是小文件，这个由应用来区分，在[创建image request](image-requests.html), 设置它的 [CacheChoice](../javadoc/reference/com/facebook/imagepipeline/request/ImageRequest.CacheChoice.html)::

```java
ImageRequest request = ImageRequest.newBuilderWithSourceUri(uri)
    .setCacheChoice(ImageRequest.CacheChoice.SMALL)
```

如果你仅仅需要一个缓存，那么不调用`setSmallImageDiskCacheConfig`即可。Image pipeline 默认会使用同一个缓存，同时`CacheChoice`也会被忽略。

### 内存用量的缩减

在 [配置Image pipeline](configure-image-pipeline.html) 时，我们可以指定每个缓存最大的内存用量。但是有时我们可能会想缩小内存用量。比如应用中有其他数据需要占用内存，不得不把图片缓存清除或者减小或者我们想检查看看手机是否已经内存不够了。

Fresco的缓存实现了[DiskTrimmable](../javadoc/reference/com/facebook/common/disk/DiskTrimmable.html) 或者 [MemoryTrimmable](../javadoc/reference/com/facebook/common/memory/MemoryTrimmable.html) 接口。这两个接口负责从各自的缓存中移除内容。

在应用中，可以给Image pipeline配置上实现了[DiskTrimmableRegistry](../javadoc/reference/com/facebook/common/disk/DiskTrimmableRegistry.html) 和 [MemoryTrimmableRegistry](../javadoc/reference/com/facebook/common/memory/MemoryTrimmableRegistry.html) 接口的对象。

实现了这两个接口的对象要持有一个列表，这个列表以[DiskTrimmable](../javadoc/reference/com/facebook/common/disk/DiskTrimmable.html) 或 [MemoryTrimmable](../javadoc/reference/com/facebook/common/memory/MemoryTrimmable.html) 对象为元素。当应用程序判定当前状态下需要削减资源使用时，这些对象才会通知列表中的[DiskTrimmable](../javadoc/reference/com/facebook/common/disk/DiskTrimmable.html) 或 [MemoryTrimmable](../javadoc/reference/com/facebook/common/memory/MemoryTrimmable.html) 元素缩减各自的资源占用。


