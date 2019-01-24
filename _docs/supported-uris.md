---
docid: supported-uris
title: 支持的 URI
layout: docs
permalink: /docs/supported-uris.html
---


Fresco 支持多种来源的图片。但是 **不支持** 相对路径的URI. 所有的 URI 都必须是绝对路径，并且带上该 URI 的 scheme。

如下：

| 类型 | Scheme | 示例 |
| ---------------- | ------- | ------------- |
| 远程图片 | `http://,` `https://` | `HttpURLConnection` 或者参考 [使用其他网络加载方案](using-other-network-layers.html) |
| 本地文件 | `file://` | `FileInputStream` | 
| Content provider | `content://` | `ContentResolver` |
| asset目录下的资源 | `asset://` | `AssetManager` |
| res目录下的资源 | `res://` | `Resources.openRawResource` |
| Uri中指定图片数据 | `data:mime/type;base64,` | 数据类型必须符合 [rfc2397规定](http://tools.ietf.org/html/rfc2397) (仅支持 UTF-8) |


<br/>
注意，只有图片资源才能使用在Image pipeline中，比如(PNG)。其他资源类型，比如字符串，或者XML Drawable在Image pipeline中没有意义。所以加载的资源不支持这些类型。像`ShapeDrawable`这样声明在XML中的drawable可能引起困惑。注意到这毕竟不是图片。如果想把这样的drawable作为图像显示，那么把这个drawable设置为[占位图](placeholder-failure-retry.html)，然后把URI设置为`null`。

### 示例：加载一个 URI

仅加载一个 URI 的示例查看 showcase 应用中的 `DraweeSimpleFragment`：
[DraweeSimpleFragment.java](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/drawee/DraweeSimpleFragment.java)

![URI 示例 ](/static/images/docs/01-using-simpledraweeview-sample.png)

### 示例： 加载本地文件

如何正确地加载用户选择的文件（比如，使用 `content://`）的例子，可以在 showcase 应用的 `DraweeMediaPickerFragment` 中查看：[DraweeMediaPickerFragment.java](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/drawee/DraweeMediaPickerFragment.java)

![本地文件示例](/static/images/docs/01-supported-uris-sample-local-file.png)

### 示例: 加载一个数据 URI

Fresco 的 showcase 应用有个 [ImageFormatDataUriFragment](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/imageformat/datauri/ImageFormatDataUriFragment.java) 类演示了如何使用占位图、失败图和重试图。

![数据 URI 示例](/static/images/docs/01-supported-uris-sample-data-uri.png)

### 更多阅读

**提示:** 你可以使用全局设置中的 *URI Override* 选项来复写 showcase 应用的许多示例代码中的图片 URI：

![复写 URI 配置](/static/images/docs/01-supported-uris-sample-override.png)
