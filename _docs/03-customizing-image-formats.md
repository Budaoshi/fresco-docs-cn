---
docid: customizing-image-formats
title: 定制图片格式
layout: docs
permalink: /docs/customizing-image-formats.html
---

通常，将图片显示到屏幕包括如下两部分：
1. 解码图片
2. 渲染解码后的图片

Fresco 允许你定制这两部分。例如：你可能需要为已存在的图片格式定制解码器或者为新的图片格式使用 Fresco 内置的渲染框架开渲染图片。或者你可能使用内置的解码器来处理解码工作然后使用定制的 Drawable 来将图片渲染到屏幕。当然，你可以同时这样做。这些定制方式可以在 Fresco 初始化时注册为全局方式，你也可以将其仅作用于选中的图片上。

解码与渲染流程看起来像下面这样子简单：
1. 从网络上或者本地磁盘中载入编码后的图片。
2. 使用图片格式校验器 `ImageFormatChecker` 来决定编码图片 `EncodedImage` 的图片格式 `ImageFormat`。Fresco 有一系列的图片格式校验器，每一种用来识别一种图片格式。
3. 根据图片格式采取适当的图片解码器 `ImageDecoder` 来解码编码后的图片并返回一个继承自 `CloseableImage` 类的对象，它表示为一个解码图片。
4. 从一系列 `DrawableFactory` 中寻找出第一个能够处理 `CloseableImage` 的对象来创建一个 `Drawable` 实例。
5. 将 4 中创建的 `Drawable` 实例渲染至屏幕。

在上述步骤 2 中，我们可以通过添加一个 `ImageFormat.FormatChecker` 来加入自定义的图片格式。你可以提供一个定制的 `ImageDecoder` 来为新的图片格式提供解码支持，或者重载内置的解码器。最后，你也可以提供一个定制好的 `DrawableFactory` 来自定义 `Drawable` 选择图片。

所有默认的图片均可以在 `DefaultImageFormats` 和 `DefaultImageFormatChecker` 类中找到，默认的 Drawable 工厂类在 `PipelineDraweeController` 中。在示例 App `Showcase` 中可以找到一些定制它们的例子。

## 定制解码器

我们以一个例子开始。创建一个定制化的解码器，只需要实现 `ImageDecoder` 接口：

```java
public class CustomDecoder implements ImageDecoder {

  @Override
  public CloseableImage decode(
      EncodedImage encodedImage,
      int length,
      QualityInfo qualityInfo,
      ImageDecodeOptions options) {
      // 解码 encodedImage 图片并返回对应的解码后的 CloseableImage
      CloseableImage closeableImage = ...;
      return closeableImage;
  }
}
```
编码后的图片可以解码为 `CloseableImage` 子类，其代表一个解码后的图片，并被系统自动缓存下来。你可以返回一个已存在的 `CloseableImage` 类型的对象，比如 `CloseableStaticBitmap`，或者定义你自己的 `CloseableImage` 类。

定制的解码器可以设置为全局，也可以作用于每一张图片。你可以按照如下方式来复写全局解码器：

```java
ImageDecoder customDecoder = ...;
Uri uri = ...;
draweeView.setController(
  Fresco.newDraweeControllerBuilder()
        .setImageRequest(
          ImageRequestBuilder.newBuilderWithSource(uri)
              .setImageDecodeOptions(
                  ImageDecodeOptions.newBuilder()
                      .setCustomImageDecoder(customDecoder)
                      .build())
              .build())
        .build());
```

**注意:** 如果你提供一个定制的解码器，它将作用于所有的图片。默认的解码器将被完全忽略。

## 定制图片格式

你可以简单的创建一个 `ImageFormat` 对象并在你的代码中持有它：

```java
private static final ImageFormat CUSTOM_FORMAT = new ImageFormat("format name", "format file extension");
```

所有支持的默认图片格式可以在 `DefaultImageFormats` 类中找到。

然后，我们需要创建一个新的格式校验器 `ImageFormat.FormatChecker` 用来识别你的图片格式。格式校验器有两个方法，一个返回头部所需要的能够决定图片格式的字节数（这个数字要尽可能的小，应该它将作用于所有图片）。另一个 `determineFormat` 方法应当返回相同的图片格式 `ImageFormat` 实例，比如下面例子中的 `CUSTOM_FORMAT`，或者如果图片是不同的格式则返回为 `null`。一个简易的格式校验器如下：

```java
public static class ColorFormatChecker implements ImageFormat.FormatChecker {

  private static final byte[] HEADER = ImageFormatCheckerUtils.asciiBytes("my_header");

  @Override
  public int getHeaderSize() {
    return HEADER.length;
  }

  @Nullable
  @Override
  public ImageFormat determineFormat(byte[] headerBytes, int headerSize) {
    if (headerSize < getHeaderSize()) {
      return null;
    }
    if (ImageFormatCheckerUtils.startsWithPattern(headerBytes, HEADER)) {
      return CUSTOM_FORMAT;
    }
    return null;
  }
}
```
定制图片格式所需的第三个部分正如上面所说的定义一个解码器，用来创建真正的解码图片。

你必须在 Fresco 初始化时，将自定义的图片格式通过 `ImageDecoderConfig` 提供给 Fresco 使用。类似的，你可以使用一个内置的图片格式复写默认的图片解码行为：

```java
ImageFormat myFormat = ...;
ImageFormat.FormatChecker myFormatChecker = ...;
ImageDecoder myDecoder = ...;
ImageDecoderConfig imageDecoderConfig = new ImageDecoderConfig.Builder()
  .addDecodingCapability(
    myFormat,
    myFormatChecker,
    myDecoder)
  .build();

ImagePipelineConfig config = ImagePipelineConfig.newBuilder()
  .setImageDecoderConfig(imageDecoderConfig)
  .build();

Fresco.initialize(context, config);
```

## 定制 drawables

如果使用 `DraweeController` 来加载图片（比如，如果你正在使用 `DraweeView`），对应的 `DrawableFactory` 被用来创建一个 drawable 来渲染基于 `CloseableImage` 的解码后的图片。如果你手动的使用 image pipeline，你必须自己处理 `CloseableImage`。

如果你使用内置类型，比如 `CloseableStaticBitmap`, `PipelineDraweeController` 已经知道如何处理图片格式，并为你创建一个 `BitmapDrawable` 对象。如果你想复写此种行为，或者为你自定义的 `CloseableImage` 提供支持，你需要实现一个 drawable 工厂类：

```java
public static class CustomDrawableFactory implements DrawableFactory {

  @Override
  public boolean supportsImageType(CloseableImage image) {
    // 你既可以复写内置的格式，比如 `CloseableStaticBitmap`
    // 也可以自己实现
    return image instanceof CustomCloseableImage;
  }

  @Nullable
  @Override
  public Drawable createDrawable(CloseableImage image) {
    // 创建并且返回你定制的 drawable，它用于确保 `CloseableImage` 是 `supportsImageType` 
    // 声明的类的实例
    CustomCloseableImage myCloseableImage = (CustomCloseableImage) image;
    Drawable myDrawable = ...; // 比如 new CustomDrawable(myCloseableImage)
    return myDrawable;
  }
}
```
要使用你自己的 drawable 工厂类，你可以复写全局或者局部的 drawable。
In order to use your drawable factory, you can either use a global or local override.

### 全局自定义 drable 复写

当 Fresco 初始化时你必须注册所有的全局 drawable 工厂类：

```java
DrawableFactory myDrawableFactory = ...;

DraweeConfig draweeConfig = DraweeConfig.newBuilder()
  .addCustomDrawableFactory(myDrawableFactory)
  .build();

Fresco.initialize(this, imagePipelineConfig, draweeConfig);
```

### 局部自定义 drable 复写

为了局部复写，`PipelineDraweeControllerBuilder` 类提供了方法来设置自定义的 drawable 工厂类：

```java
DrawableFactory myDrawableFactory = ...;
Uri uri = ...;

simpleDraweeView.setController(Fresco.newDraweeControllerBuilder()
  .setUri(uri)
  .setCustomDrawableFactory(factory)
  .build());
```
