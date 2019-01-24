---
docid: index
title: Fresco 入门指南
layout: docs
permalink: /docs/index.html
---

本指南一步步引导你如何在你的应用中使用 Fresco，包括加载你的第一张图片。

### 1. 更新 Gradle 配置

编辑 你的 `build.gradle` 文件，你必须将以下部分加入到 `dependencies` :

```groovy
dependencies {
  // 其他依赖
  implementation 'com.facebook.fresco:fresco:{{site.current_version}}'
}
```

下面的依赖需要根据需求添加：

```groovy
dependencies {
  
  // 支持 GIF 动图，需要添加
  implementation 'com.facebook.fresco:animated-gif:{{site.current_version}}'

  // 支持 WebP （静态图+动图），需要添加
  implementation 'com.facebook.fresco:animated-webp:{{site.current_version}}'
  implementation 'com.facebook.fresco:webpsupport:{{site.current_version}}'

  // 仅支持 WebP 静态图，需要添加
  implementation 'com.facebook.fresco:webpsupport:{{site.current_version}}'

  // 添加 Android support 支持库（你或许已经有了这个或者相似的依赖）
  implementation 'com.android.support:support-core-utils:{{site.support_library_version}}'
}
```

### 2. 初始化 Fresco & 声明权限

Fresco 需要先初始化。你应当仅初始化一次，所以在你的 Application 中初始化是个不错的选择。

```java
[MyApplication.java]
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        Fresco.initialize(this);
    }
}
```

*注意:* 记得在你的 ```AndroidManifest.xml``` 中声明 Application 类和添加必要的权限。大部分情况你需要 INTERNET 权限。

```xml
  <manifest
    ...
    >
    <uses-permission android:name="android.permission.INTERNET" />
    <application
      ...
      android:label="@string/app_name"
      android:name=".MyApplication"
      >
      ...
    </application>
    ...
  </manifest>
```

### 3. 创建布局文件

在你的 XML 布局文件中，于顶层元素添加一个自定义的命名空间（namespace）来访问自定义的 `fresco:` 属性，这些属性用来控制图片的加载与显示。

```xml
<!-- Any valid element will do here -->
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:fresco="http://schemas.android.com/apk/res-auto"
    android:layout_height="match_parent"
    android:layout_width="match_parent"
    >
```

然后添加在布局中添加 ```SimpleDraweeView``` :

```xml
<com.facebook.drawee.view.SimpleDraweeView
    android:id="@+id/my_image_view"
    android:layout_width="130dp"
    android:layout_height="130dp"
    fresco:placeholderImage="@drawable/my_drawable"
    />
```

你只需要按照下面的方式来显示一张图片:

```java
Uri uri = Uri.parse("https://raw.githubusercontent.com/facebook/fresco/master/docs/static/logo.png");
SimpleDraweeView draweeView = (SimpleDraweeView) findViewById(R.id.my_image_view);
draweeView.setImageURI(uri);
```
接下来的工作交给 Fresco 处理。

在目标图片准备好之前，占位图会一直显示。目标图片会被下载、缓存、展示。当控件脱离屏幕后，Fresco 会将其从内存清除。



