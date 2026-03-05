---
title: Android设备中dp和px的换算关系
date: 2021-09-05 13:55:00
comments: true
tags:
- Android
- App开发
- 设备适配
- 屏幕适配
categories:
- 软件开发
- 小程序和APP
---


Android项目开发中，对于layout布局文件中的控件，我们通常会使用dp作为单位，而不是px。这是因为dp是一个密度无关的单位，而px是一个物理像素单位。在不同的设备上，px的大小是不同的，而dp的大小是不变的。因此，使用dp作为单位可以确保在不同设备上的显示效果是一致的。

正确理解dp和px的换算关系，可以让我们的Android项目的设备适配更加准确。


## dp 和 px 的换算关系

### 基本换算公式

```
px = dp * (dpi / 160)
```


其中：
- `dp` 是密度无关像素 (density-independent pixels)
- `px` 是物理像素 (pixels)
- `dpi` 是屏幕密度 (dots per inch)

### Android 密度分类

Android 定义了几种标准密度：
- `ldpi`: 120 dpi
- `mdpi`: 160 dpi (基准密度)
- `hdpi`: 240 dpi
- `xhdpi`: 320 dpi
- `xxhdpi`: 480 dpi
- `xxxhdpi`: 640 dpi

### 换算示例

在不同密度屏幕上，100dp 对应的像素数：
- mdpi (160dpi): 100px
- hdpi (240dpi): 150px
- xhdpi (320dpi): 200px
- xxhdpi (480dpi): 300px

## Android 屏幕宽度（以 dp 为单位）

### 屏幕宽度分类

Android 将屏幕宽度分为几类：
- `small`: 至少 320dp
- `normal`: 至少 320dp
- `large`: 至少 480dp
- `xlarge`: 至少 720dp

### 常见设备屏幕宽度（dp）

- 大多数手机: 320dp - 480dp
- 平板电脑: 600dp - 1200dp+
- 7英寸平板: 约 600dp
- 10英寸平板: 约 800dp 或更高

### 获取屏幕宽度的方法

在代码中可以通过以下方式获取屏幕宽度：

```java
DisplayMetrics metrics = getResources().getDisplayMetrics();
float screenWidthDp = metrics.widthPixels / metrics.density;
```


使用 dp 作为单位可以确保 UI 在不同密度的设备上保持一致的物理尺寸。


![Android设备中dp和px的换算](https://s2.loli.net/2025/08/26/yUlB2pMQCzI9Lgb.jpg)


## 理解 mdpi 是 160dpi

简而言之：每英寸的屏幕大小中有160个像素点，其他的hdpi等以此类推，因此，这是设备的硬件属性。

### 基本概念

- **mdpi** (medium density dots per inch) 是 Android 定义的一个标准屏幕密度等级
- **160dpi** 意味着每英寸包含 160 个像素点
- 这是 Android 系统设定的基准密度，所有其他密度都相对于这个值进行计算

### 为什么选择 160dpi 作为基准

1. **历史原因**: 160dpi 在早期 Android 设备中是一个常见的屏幕密度
2. **计算便利**: 以 160 为基准，其他密度都是它的整数倍或简单分数：
   - ldpi: 120dpi (0.75 × 160)
   - hdpi: 240dpi (1.5 × 160)
   - xhdpi: 320dpi (2 × 160)
   - xxhdpi: 480dpi (3 × 160)

### 实际意义

当屏幕密度为 160dpi 时：
- 1dp = 1px（在 mdpi 屏幕上，密度无关像素等于物理像素）
- 这简化了开发过程中的计算

### dp 与物理尺寸的关系

由于 160dpi 的定义，可以得出：
- 1 英寸 = 160 像素（在 mdpi 屏幕上）
- 1dp ≈ 1/160 英寸（在 mdpi 屏幕上的物理尺寸）

这就是为什么 mdpi 被选作基准密度的原因，它使得 dp 单位在该密度屏幕上直接对应到像素，便于理解和计算。
