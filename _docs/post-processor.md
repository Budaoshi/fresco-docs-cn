---
docid: post-processor
title: 改变图片 (后处理器)
layout: docs
redirect_from: /docs/post-processor.html
permalink: /docs/modifying-image.html
---

### 动机

后处理器（Post-processors）允许你对请求到的图片进行修改。大多数情况下，图片在发送到客户端之前，应该在服务器端做好了处理，因为移动设备资源受限。然而，也有很多情况，在客户端处理是个不错的选择。比方说，图片存储在第三方服务器，你对其没有控制权，又或者图片是张本地图片（在设备中）。

### 后台处理

在 Fresco 的 pipeline中，后处理器在后期才会使用到，此时图片已经解码成了 bitmap，最初的版本也已经放置到了内存 Bitmap 缓存中。它可以直接作用于 Bitmap 对象，也可以使用不同的尺寸来创建新的 Bitmap。

理想情况下，后处理器应当为给定的参数提供缓存键（cache key）。这样做的好处是，新创建的 bitmap 也可以在内存中缓存而不需要每次都重建。

所有的后处理都有后台线程执行。然而，稚嫩的迭代或者复杂计算仍旧会花费很长时间，这应该要避免。如果你需要花费非线性的时间来计算一批像素的化，这里有个小节为你提供了一些建议，你可以使用稚嫩的代码来缩短后处理器的处理时间。

### 示例: 创建一个灰度模式（Grey-Scale）的过滤器

让我们从一个简单的示例开始：创建一个后处理器，可以将 bitmap 转化成灰度级的图片。为此，我们需要迭代 bitmap 的所有像素值并替换它们的色值。

在交给后处理器前，我们复制一份图片。这样，在你后处理过程中不会改变缓存中原始的图片。在 Android 5.0 之前，副本与原始图一样，不在 Java 堆中保存。

`BasePostprocessor` 期望子类复写其中的一个 `BasePostprocessor#process` 方法。最简单的方法就是原地对 bitmap 进行修改。因此，在你后处理过程中缓存中原始的图片不会受到影响。之后我们会讨论如何改变输出 bitmap 的配置信息以及尺寸大小。

```java
public class FastGreyScalePostprocessor extends BasePostprocessor {

  @Override
  public void process(Bitmap bitmap) {
    final int w = bitmap.getWidth();
    final int h = bitmap.getHeight();
    final int[] pixels = new int[w * h];

    bitmap.getPixels(pixels, 0, w, 0, 0, w, h);

    for (int x = 0; x < w; x++) {
      for (int y = 0; y < h; y++) {
        final int offset = y * w + x;
        pixels[offset] = getGreyColor(pixels[offset]);
      }
    }

    // this is much faster then calling #getPixel and #setPixel as it crosses
    // the JNI barrier only once
    bitmap.setPixels(pixels, 0, w, 0, 0, w, h);
  }

  static int getGreyColor(int color) {
    final int alpha = color & 0xFF000000;
    final int r = (color >> 16) & 0xFF;
    final int g = (color >> 8) & 0xFF;
    final int b = color & 0xFF;

    // see: https://en.wikipedia.org/wiki/Relative_luminance
    final int luminance = (int) (0.2126 * r + 0.7152 * g + 0.0722 * b);

    return alpha | luminance << 16 | luminance << 8 | luminance;
  }
}
```

![Showcase app with grey-scale filter](/static/images/docs/02-post-processor-grey.png)

### 缓存后处理后的结果

我们已经看到了，后处理器的计算需要消耗大量资源，我们想缓存后期的处理结果。输出的 bitmaps 缓存位置与解码后的输入图片一样。

为了使用这个特性，后处理器必须复写 `PostProcessor#getPostProcessorCacheKey` 方法。它应该返回一个独立于所有影响到修改的输入值的缓存键。

例如，我们继承一个已存在的 `WatermarkPostprocessor` 来在图片上多次绘制文字水印：

```java
public class CachedWatermarkPostprocessor extends WatermarkPostprocessor {

  @Override
  public CacheKey getPostprocessorCacheKey() {
    return new SimpleCacheKey(String.format(
        (Locale) null,
        "text=%s,count=%d",
        mWatermarkText,
        mCount));
  }
}
```

### 进阶: JNI and 模糊处理

有关后处理器，最常问到就是模糊处理的效果。幸运的是，Fresco 采用本地 C 代码实现了一个高效的模糊效果，可以通过 `NativeBlurFilter#iterativeBoxBlur` 访问。

在你考虑采用更高级的后处理器时，采用本地代码实现是一种提高性能的不错方法。如果你依照此建议，看看 `blur_filter.c` 是如果使用本地代码来操作 bitmaps 的。最要的是它向你诠释了如何锁定内存中的像素已经其它一些重要的技巧。

![Showcase app with blur post-processor](/static/images/docs/02-post-processor-blur.png)

### 进阶: 改变 Bitmap 大小

即便使用本地代码完成了一个高效的处理，后处理器仍然可能会是个耗时操作。为了更高效的处理模糊效果，我们可以缩减图片大小，对更小的图片进行模糊操作。在显示时让 GPU 来重新放大图片。由于模糊后的图片没有确定的边界，这种优化方式通常会哪些辨认。

在我们新版本后处理器中，我们重写了一个 `BasePostProcessor#process()` 的重载版本。这个版本提供了一个 `PlatformBitmapFactory` 对象，我们可以使用该对象来自定义输出 bitmap。要注意的是，我们不能再改变 `sourceBitmap`，因为它不是我们之前创建图片的副本。

```java
public class ScalingBlurPostprocessor extends FullResolutionBlurPostprocessor {

 /**
   * A scale ration of 4 means that we reduce the total number of pixels to process by factor 16.
   */
  private static final int SCALE_RATIO = 4;

  @Override
  public CloseableReference<Bitmap> process(
      Bitmap sourceBitmap,
      PlatformBitmapFactory bitmapFactory) {
    final CloseableReference<Bitmap> bitmapRef = bitmapFactory.createBitmap(
        sourceBitmap.getWidth() / SCALE_RATIO,
        sourceBitmap.getHeight() / SCALE_RATIO);

    try {
      final Bitmap destBitmap = bitmapRef.get();
      final Canvas canvas = new Canvas(destBitmap);

      canvas.drawBitmap(
          sourceBitmap,
          null,
          new Rect(0, 0, destBitmap.getWidth(), destBitmap.getHeight()),
          mPaint);

      NativeBlurFilter.iterativeBoxBlur(destBitmap, BLUR_RADIUS / SCALE_RATIO, BLUR_ITERATIONS);

      return CloseableReference.cloneOrNull(bitmapRef);
    } finally {
      CloseableReference.closeSafely(bitmapRef);
    }
  }
}
```

![Showcase app with scaling blur post-processor](/static/images/docs/02-post-processor-scaling-blur.png)

### 约束

在创建后处理器时请记得如下规则：

* 如果要反复展示同一张图片，你必须在每次请求时都要指定后处理器。对于同一张图片，你可以随意的对不同请求采用不同的后处理器。
* 后处理器目前还不支持[动图](animations.html)。
* 若要在后处理器中使用透明度，你可以调用 `destinationBitmap.setHasAlpha(true);` 方法。
* **不要** 复写多个 `process` 方法。若要这样做可能会出现不可预期的结果。
* **不要** 在使用  `process` 方法需要创建新的尺寸 bitmap 时改变原图。
* **不要** 持有 bitmap 对象的引用。它们在 image pipeline 有着自己的内存管理。通常 destBitmap 在你的 Drawee 或者 DataSource 中会自动清除。
* **不要** 使用 Android 自带的 `Bitmap.createBitmap` 方法来创建新的 Bitmap。它会与 Fresco 中 Bitmap 池里的冲突。

### 完整示例

查看 showcase 应用中的 `ImagePipelinePostProcessorFragment` 来了解完整示例： [ImagePipelinePostProcessorFragment.java](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/imagepipeline/ImagePipelinePostProcessorFragment.java)。它包括了本节中涉及到的所有后处理器，以及其它的处理器。
