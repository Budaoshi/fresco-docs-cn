---
docid: proguard
title: 用 Fresco 装载(Ship)你的应用
layout: docs
permalink: /docs/proguard.html
---

Fresco的体积可能有点庞大，但这不会让你的应用变得很大。我们强烈推荐你使用 ProGuard 工具，同时构建多 APKs 来让你的应用保持小体量。

### ProGuard

1.9.0 版本后的 Fresco 自带了一个 ProGuard 配置文件，在你开启 ProGuard 时会自动应用这些规则。
为了使能 ProGuard，修改你的 `build.gradle`文件，在 `release` 节加入下面几行：

```groovy
android {
  buildTypes {
    release {
      minifyEnabled true
      proguardFiles getDefaultProguardFile('proguard-android.txt')
    }
  }
}
```

### 构建多 APKs

Fresco 大部分是用 Java 写的，但也存在少量的 C++ 代码。C++ 代码被编译成了不同 CPU 类型（称为 “ABIs”）的版本。当前，Fresco 支持 5 类 ABIs。

1. `armeabiv-v7a`: 版本 7 或更高版本的 ARM 处理器。大多数 2011-15 发行的 Android 手机使用的是这类处理器。
2. `arm64-v8a`: 64-bit ARM 处理器。能够在新发行的设备中看到，比如 Samsung Galaxy S6。
3. `x86`: 大部分的平板手机和模拟器使用这类处理器。
4. `x86_64`: 用在 64-bit 的平板上。

Fresco 的二进制版本中有这 5 类平台的本地 `.so` 文件。你可以为不同处理器类型的系统创建独立的 APKs 来缩减你的应用大小。

如果你的应用不支持 Android 2.3(Gingerbread)，你就不需要 `armeabi` 版本的文件。

为构建多 APK是，在你的 `build.gradle` 文件的 `android` 节加入 `splits` 小节。

```groovy
android {
  // rest of your app's logic
  splits {
    abi {
        enable true
        reset()
        include 'x86', 'x86_64', 'arm64-v8a', 'armeabi-v7a'
        universalApk false
    }
  }
}
```

查看 [Android 发布文档](https://developer.android.com/google/play/publishing/multiple-apks.html)了解更多有关多 APKs 工作原理的内容。
