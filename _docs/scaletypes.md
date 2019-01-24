---
docid: scaletypes
title: 缩放类型
layout: docs
permalink: /docs/scaletypes.html
---

你可以为你 Drawee 中不同的 drawables 指定不同的缩放类型。

### 可用的缩放类型

| 缩放类型            | 保持宽高比          |总是填满 View        |执行缩放           |说明         |
| ---------         | :-:                | :-:               | :-:              | ----------- |
| center            | ✓                  |                   |                  | 居中，无缩放。 |
| centerCrop        | ✓                  |  ✓                |  ✓               | 保持宽高比缩小或放大，使得两边都大于或等于显示边界<br/>且宽或高契合显示边界。<br/> 宽或者高固定 <br/>居中显示。|
| focusCrop         | ✓                  |  ✓                |  ✓               | 同 centerCrop, 但居中点不是中点，而是指定的某个点。|
| centerInside      | ✓                  |                   |  ✓               | 缩放图片使两边都在显示边界内，居中显示。和 `fitCenter` 不同，不会对图片进行放大。<br/>宽高比固定<br>如果图尺寸大于显示边界，则保持宽高比缩小图片。|
| fitCenter         | ✓                  |                   |  ✓               | 保持宽高比，缩小或者放大，使得图片完全显示在显示边界内，且宽或高契合显示边界。居中显示。|
| fitStart          | ✓                  |                   |  ✓               | 同上。但不居中，和显示边界左上对齐。|
| fitEnd            | ✓                  |                   |  ✓               | 同fitCenter， 但不居中，和显示边界右下对齐。|
| fitXY             |                    |  ✓                |                  | 不保存宽高比，填充满显示边界。|
| none              | ✓                  |                   |                  | 用于 Android 的 title 模式。|

这些缩放类型和Android [ImageView](http://developer.android.com/reference/android/widget/ImageView.ScaleType.html) 支持的缩放类型几乎一样.唯一不支持的缩放类型是 `matrix`。Fresco 提供了 `focusCrop` 作为补充，通常这个使用效果更佳。

### 如何设置缩放类型

目标图片，占位图，重试图和失败图都可以在 xml 中进行设置，用 `fresco:actualImageScaleType` 这样的属性。你也可以使用 [GenericDraweeHierarchyBuilder](../javadoc/reference/com/facebook/drawee/generic/GenericDraweeHierarchyBuilder.html) 类在代码中进行设置。

即使 hierarchy 已经构建完成，目标图片的缩放类型仍然可以通过 [GenericDraweeHierarchy](../javadoc/reference/com/facebook/drawee/generic/GenericDraweeHierarchy.html) 类在运行中进行修改。

然后，不要使用 `android:scaleType` 属性，也不要使用 `setScaleType()` 方法，它们对 Drawees 无效。

### 缩放类型："focusCrop"

Android 和 Fresco 都提供了 `centerCrop`缩放类型。它会保持长宽比，放大或缩小图片，填充满显示边界，居中显示。

这个缩放类型在通常情况下很有用。但是裁剪效果通过在你需要的时候不会生效。以人脸图片为例，如果你想从右下角裁剪图片，`centerCrop` 会出错。

通过指定一个焦点，你可以说图片的哪部分可以居中显示在视图中。如果你指定的焦点位于图片顶部，比如（0.5f, 0），不论如何，我们保证这个焦点可以被看到并且尽可能地让其居中显示。

焦点是以相对方式给出的，比如 (0f, 0f) 是左上对齐显示，(1f, 1f) 是右下角对齐。相对坐标使得居中点位置和具体尺寸无关，这是非常实用的。

(0.5f, 0.5f) 的居中点位置和缩放类型 `centerCrop` 是等价的。 

如果要使用此缩放模式，首先在 XML 中指定缩放模式:

```xml
  fresco:actualImageScaleType="focusCrop"
```

在Java代码中，给你的图片指定居中点：

```java
PointF focusPoint = new PointF(0f, 0.5f);
mSimpleDraweeView
    .getHierarchy()
    .setActualImageFocusPoint(focusPoint);
```


### 缩放类型： none

如果你要使用tile mode进行显示，那么需要将scale type 设置为none.

### 缩放类型：自定义 SacleType

有时候现有的 ScaleType 不符合你的需求，我们允许你通过实现 `ScalingUtils.ScaleType` 来拓展它，这个接口里面只有一个方法：`getTransform`，它会基于以下参数来计算转换矩阵：

* parent bounds (View 坐标系中对图片的限制区域)
* child size （目标图片的高宽）
* focus point （子坐标系中的相对坐标）

当然，你的类里面应该包含了你需要额外信息。

我们来看一个例子，假设 View 应用了一些 padding， `parentBounds` 为 `(100, 150, 500, 450)`， 图片大小为`(420,210)`。那么我们知道 View 的宽度度为 `500 - 100 = 400`， 高度为 `450 - 150 = 300`。那么如果我们不做任何处理，图片在 View 坐标系中就会被画在`(0, 0, 420, 210)`。但是 `ScaleTypeDrawable` 会使用 `parentBounds` 来进行限制，那么画布就会被 `(100, 150, 500, 450)` 这块矩阵裁切，那么最后图片显示区域就是 `(100, 150, 420, 210)`。

为了避免这种情况，我们可以变换 `(parentBounds.left, parentBounds.top)` （在这个例子中是`(100, 150)`）。现在图片比 View 还宽，现在无论是将图片放置在 `(100, 150, 500, 360)` 还是 `(0, 0, 400, 210)`，我们都会损失右侧20像素的宽度。

那么我们可以将它缩小一点（`400/420`的缩放比例），让它能够放置在 View 给它分配的区域中(缩放后变成了`400, 200`)。那么现在宽度刚刚好，但是高度却不是了。

我们需要进一步处理，我们可以算一下高度还剩余的空间 `300 - 200 = 100`。我们可以将他们平均分配在上下两侧，这样图片就被居中了，真棒！你可以通过实现 `FIT_CENTER` 来让它做到这点。那么来看代码吧：

```java
  public  class AbstractScaleType implements ScaleType {
    @Override
    public Matrix getTransform(Matrix outTransform, Rect parentRect, int childWidth, int childHeight, float focusX, float focusY) {
      // 取宽度和高度需要缩放的倍数中最小的一个
      final float scaleX = (float) parentRect.width() / (float) childWidth;
      final float scaleY = (float) parentRect.height() / (float) childHeight;
      final float scale = Math.min(scaleX, scaleY);
      
      // 计算为了均分空白区域，需要偏移的x、y方向的距离
      final float dx = parentRect.left + (parentRect.width() - childWidth * scale) * 0.5f;
      final float dy = parentRect.top + (parentRect.height() - childHeight * scale) * 0.5f;
      
      // 最后我们应用它
      outTransform.setScale(scale, scale);
      outTransform.postTranslate((int) (dx + 0.5f), (int) (dy + 0.5f));
      return outTransform;
    }
  }
```

### 完整示例

showcase 应用中的 `DraweeScaleTypeFragment` 来了解完整示例：[DraweeScaleTypeFragment.java](https://github.com/facebook/fresco/blob/master/samples/showcase/src/main/java/com/facebook/fresco/samples/showcase/drawee/DraweeScaleTypeFragment.java)

![带有缩放类型示例的 Showcase 应用](/static/images/docs/01-scaletypes-sample-1.png)

![带有缩放类型示例的 Showcase 应用](/static/images/docs/01-scaletypes-sample-2.png)

