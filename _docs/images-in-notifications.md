---
docid: images-in-notifications
title: 通知栏中的图片
layout: docs
permalink: /docs/images-in-notifications.html
---

如果需要在通知栏显示图片，你可以使用 `BaseBitmapDataSubscriber` 从 `ImagePipeline` 中获取图片。这种方式会在 `NotificationManager#notify` 方法调用时，由系统打包(parcel) 后安全地传递给通知。本节将给出一个完整示例来阐述实现过程。

### 步骤

首先 使用 URI 创建一个 `ImageRequest` 图片请求：

```java
ImageRequest imageRequest = ImageRequest.fromUri("http://example.org/user/42/profile.jpg"));
```

然后创建一个`DataSource` 对象引用并从 `ImagePipeline` 获取解码图片：

```java
ImagePipeline imagePipeline = Fresco.getImagePipeline();
DataSource<CloseableReference<CloseableImage>> dataSource = imagePipeline.fetchDecodedImage(imageRequest, null);
```

`DataSource` 与 `Future` 很类似, 我们需要添加一个 `DataSubscriber` 来处理结果。`BaseBitmapDataSubscriber` 类抽象了一些处理 `Bitmap` 的复杂方法：

```java
dataSource.subscribe(
    new BaseBitmapDataSubscriber() {

      @Override
      protected void onNewResultImpl(Bitmap bitmap) {
        displayNotification(bitmap);
      }

      @Override
      protected void onFailureImpl(DataSource<CloseableReference<CloseableImage>> dataSource) {
        // In general, failing to fetch the image should not keep us from displaying the
        // notification. We proceed without the bitmap.
        displayNotification(null);
      }
    },
    UiThreadImmediateExecutorService.getInstance());
}
```

在 Android 上，`displayNotification(Bitmap)` 方法 与常规的实现方式类似：

```java
private void displayNotification(@Nullable Bitmap bitmap) {
  final NotificationCompat.Builder notificationBuilder =
      new NotificationCompat.Builder(getContext())
          .setSmallIcon(R.drawable.ic_done)
          .setLargeIcon(bitmap)
          .setContentTitle("Fresco Says Hello")
          .setContentText("Notification Text ...");

  final NotificationManager notificationManager =
      (NotificationManager) getContext().getSystemService(Context.NOTIFICATION_SERVICE);

  notificationManager.notify(NOTIFICATION_ID, notificationBuilder.build());
}
```

### 完整示例

可以查看 showcase 应用中的 `ImagePipelineNotificationFragment` 来了解完整示例：[ImagePipelineNotificationFragment.java](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/imagepipeline/ImagePipelineNotificationFragment.java)

![Showcase app with a notification](/static/images/docs/02-images-in-notifications-sample.png)
