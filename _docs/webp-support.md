---
docid: webp-support
title: Webp 支持
layout: docs
permalink: /docs/webp-support.html
---

[WebP](https://en.wikipedia.org/wiki/WebP) 是一种图片格式，支持有损和无损压缩。此外，它也支持透明度和动画。


### Android 中的支持

Android 系统在 4.0 版本中添加入了 WebP 的支持，并在 4.2.1 版本中加强了它:

* 4.0+ (Ice Cream Sandwich): 基础的 WebP 支持
* 4.2.1+ (Jelly Beam MR1): 支持带透明度与无损的 WebP

通过加入 Fresco 的 webpsupport 模块，应用可以在所有的 Android 系统上显示所有类型的 WebP 图片：

|  配置         | 基础 WebP  | 无损或者透明 WebP | 动画 WebP  |
|---                     |:-:          |:-:                           |:-:             |
| OS < 4.0               |             |                              |                |
| OS >= 4.0              | ✓           |                              |                |
| OS >= 4.2.1            | ✓           | ✓                            |                |
| Any OS + webpsupport   | ✓           | ✓                            |                |
| Any OS + animated-webp | ✓           | (✓ webpsupport 或者 OS >= 4.2.1)           |  ✓             |


### 在老版本上添加对静态 WebP 图片的支持

你唯一需要做的事情就是将 `webpsupport` 库加入到你的依赖中。它支持所有类型的静态 WebP 图片。比如，你可以用它在 Gingerbread 系统上显示透明的 WebP 图片。

```groovy
dependencies {
  // ... your app's other dependencies
  implementation 'com.facebook.fresco:webpsupport:{{site.current_version}}'
}
```

### 动态 WebP（Animated WebP）

为了能够显示动态的 WebP 图片，你必须加入下面的依赖：

```groovy
dependencies {
  // ... your app's other dependencies
  implementation 'com.facebook.fresco:animated-webp:{{site.current_version}}'
  implementation 'com.facebook.fresco:webpsupport:{{site.current_version}}'
}
```
