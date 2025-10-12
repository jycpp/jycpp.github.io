---
title: 理解Android开发中的Context和Activity
date: 2024-07-20 14:35:22
comments: true
tags:
- Android
- Context
- Activity
- 权限请求
- Kotlin
- 多态性
- 继承关系
- AppCompatActivity
- 类型转换
- 权限管理
categories:
- 软件开发
- 小程序和APP
---



在Android系统开发中，Context是一个非常重要的概念，它提供了访问应用程序环境的方法和资源。Activity是Context的一个具体实现，它代表了一个用户界面的活动。如果搞不清楚这两个概念，很多代码就会看得很困惑。


我们以这段代码为例，需要实现动态请求用户授权的功能，为什么这里的 this，既可以代表 context，也可以用作 Activity 类型的变量？

```kotlin
class MainActivity : AppCompatActivity(), UiHandler.UiEventListener {

    companion object {
        const val PERMISSION_REQUEST_CODE_FOR_ALBUM = 1001
        const val PERMISSION_REQUEST_CODE_FOR_CAMERA = 1002
    }

    private fun checkAndRequestPermissionsForAlbum(): Boolean {
        // Build.VERSION_CODES.R = 31，Android 11
        return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            // Android 11 及以上版本，不再使用
            if (!Environment.isExternalStorageManager()) {
                // 请求所有文件访问权限
                val intent = Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION)
                intent.data = Uri.parse("package:$packageName")
                startActivity(intent)
                false
            } else {
                true
            }
        } else {
            // Android 10 及以下版本
            if (ContextCompat.checkSelfPermission(
                    this,
                    Manifest.permission.READ_EXTERNAL_STORAGE
                ) != PackageManager.PERMISSION_GRANTED ||
                ContextCompat.checkSelfPermission(
                    this,
                    Manifest.permission.WRITE_EXTERNAL_STORAGE
                ) != PackageManager.PERMISSION_GRANTED
            ) {
                ActivityCompat.requestPermissions(
                    this,
                    arrayOf(
                        Manifest.permission.READ_EXTERNAL_STORAGE,
                        Manifest.permission.WRITE_EXTERNAL_STORAGE
                    ),
                    PERMISSION_REQUEST_CODE_FOR_ALBUM
                )
                false
            } else {
                true
            }
        }
    }

}
```


要理解以上代码，需要了解涉及到 Kotlin 和 Android 中的类型系统和继承关系的知识。

## 过程分析

### 1. 继承关系

在 Android 中，`Activity` 类的继承链是这样的：

```
Object
  ↓
Context
  ↓
ContextWrapper
  ↓
ContextThemeWrapper
  ↓
Activity
  ↓
ComponentActivity
  ↓
FragmentActivity
  ↓
AppCompatActivity
```


### 2. 多态性

由于 `AppCompatActivity`（您在 [MainActivity.kt]中继承的类）是 `Activity` 的子类，同时也是 `Context` 的子类，所以它具有两者的特性：

```kotlin
class MainActivity : AppCompatActivity() {
    // this 的类型实际上是 AppCompatActivity，但同时也是：
    // - Activity 类型（因为 AppCompatActivity 继承自 Activity）
    // - Context 类型（因为 Activity 继承自 Context）
}
```


### 3. 方法参数的类型兼容性

让我们看几个例子：

#### 接受 Context 参数的方法：
```kotlin
// ContextCompat.checkSelfPermission 需要 Context 类型
ContextCompat.checkSelfPermission(
    this,  // this 是 Context 类型，完全匹配
    Manifest.permission.READ_EXTERNAL_STORAGE
)
```


#### 接受 Activity 参数的方法：
```kotlin
// ActivityCompat.requestPermissions 需要 Activity 类型
ActivityCompat.requestPermissions(
    this,  // this 是 Activity 类型，完全匹配
    arrayOf(...),
    PERMISSION_REQUEST_CODE_FOR_ALBUM
)
```


#### 启动 Activity：
```kotlin
// startActivity 是 Activity 类的方法
startActivity(intent)  // this 是 Activity，可以调用 startActivity
```


### 4. 在代码中的应用

在我们的 [checkAndRequestPermissionsForAlbum]方法中：

```kotlin
private fun checkAndRequestPermissionsForAlbum(): Boolean {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
        // ...
        startActivity(intent)  // 1. 调用 Activity 的 startActivity 方法
    } else {
        // ...
        ContextCompat.checkSelfPermission(
            this,  // 2. this 作为 Context 传递
            Manifest.permission.READ_EXTERNAL_STORAGE
        )
        // ...
        ActivityCompat.requestPermissions(
            this,  // 3. this 作为 Activity 传递
            // ...
        )
    }
}
```


### 5. 多重身份的示例

```kotlin
// this 作为 Activity 使用
this.startActivity(intent)
this.finish()
this.setContentView(R.layout.activity_main)

// this 作为 Context 使用
val sharedPrefs = this.getSharedPreferences("prefs", Context.MODE_PRIVATE)
val color = ContextCompat.getColor(this, R.color.primary)
val drawable = ContextCompat.getDrawable(this, R.drawable.ic_launcher)

// this 作为 AppCompatActivity 使用
this.supportActionBar?.setTitle("Title")
```


### 6. 类型转换说明

在 Kotlin 中，子类对象可以自动向上转换为父类类型，无需显式转换：

```kotlin
val activity: Activity = this  // 自动向上转换
val context: Context = this    // 自动向上转换
```


这就是为什么在您的代码中，同一个 `this` 可以在不同场景下被视为 `Activity` 或 `Context` 的原因。这是面向对象编程中多态性的体现，让代码更加灵活和简洁。


## 主要结论

回到开头的代码，扩展一下，如果我们将当前的 Activity 传给了其他 class，这些 class 主要是用来分担主 Activity 的功能，通过我们会将当前 Activity 的 this 指针作为参数传进去，我们这里举一个名字为 MyPhotoAlbum的class 的例子，来说明在这个 class 中完成权限检查和请求的过程，主要看以下代码中 this 变量的替换：

```kotlin
class PhotoAlbumManager(context: Context) {
    private val context: Context? = context
    private val contentResolver: ContentResolver = context.contentResolver

    private fun checkAndRequestPermissionsForAlbum(): Boolean {
        return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            // Android 11 及以上版本
            if (!Environment.isExternalStorageManager()) {
                // 请求所有文件访问权限
                val intent = Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION)
                intent.data = Uri.parse("package:${context.packageName}")
                context.startActivity(intent)
                false
            } else {
                true
            }
        } else {
            // Android 10 及以下版本
            if (ContextCompat.checkSelfPermission(
                    context,
                    Manifest.permission.READ_EXTERNAL_STORAGE
                ) != PackageManager.PERMISSION_GRANTED ||
                ContextCompat.checkSelfPermission(
                    context,
                    Manifest.permission.WRITE_EXTERNAL_STORAGE
                ) != PackageManager.PERMISSION_GRANTED
            ) {
                // 确保 context 是 Activity 类型再请求权限
                val activity = context as? Activity
                if (activity == null) {
                    Log.e(TAG, "Context is not an Activity, cannot request permissions")
                    return false
                }

                ActivityCompat.requestPermissions(
                    activity,
                    arrayOf(
                        Manifest.permission.READ_EXTERNAL_STORAGE,
                        Manifest.permission.WRITE_EXTERNAL_STORAGE
                    ),
                    PERMISSION_REQUEST_CODE_FOR_ALBUM
                )
                false
            } else {
                true
            }
        }
    }

    companion object {
        private const val TAG = "PhotoAlbumManager"
        private const val PERMISSION_REQUEST_CODE_FOR_ALBUM = 1001
    }
}

```

以上代码中，我们将主界面的Activity做为context类型的参数传进去，然后在这个class中完成权限检查和请求的过程，如果context类型不能满足要求的时候，我们可以通过“val activity = context as? Activity” 将其转为Activity来完成权限请求。





