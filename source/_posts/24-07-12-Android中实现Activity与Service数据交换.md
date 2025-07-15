---
title: Android中实现Activity与Service数据交换
date: 2024-07-12 20:32:56
comments: true
tags:
- Android
- Activity
- Service
- LiveData
- ViewModel
- EventBus
- 数据交换
- 后台服务
- 实时更新
- 架构组件
categories:
- 软件开发
- 小程序和APP
---



> 核心要点： 
> 1. LiveData是kotlin中特有的机制，可以被observe，通过ViewModel可以实现线程安全，更稳定。
> 2. Application的子类，通过在AndroidManifest.xml中的application标签中配置android:name属性，指定为自定义的Application类，可以实现全局传递变量。
> 3. 在Service中可以通过Application的实例获取ViewModel，从而更新数据；在activity中可以通过ViewModel的实例获取LiveData，从而实时更新UI。




在 Android 中实现后台服务与 Activity 之间的实时数据更新，有几种高效可靠的方法。以下是针对您场景的最佳实现方案：

### 推荐方案：使用 LiveData + ViewModel（Android 架构组件）

#### 1. 创建 DeviceInfoViewModel
位置：`app/src/main/java/cn/xiaoa/app/smartdevice/viewmodel/DeviceInfoViewModel.kt`

```kotlin
package cn.xiaoa.app.smartdevice.viewmodel

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import cn.xiaoa.app.smartdevice.model.DeviceInfo

class DeviceInfoViewModel : ViewModel() {
    private val _deviceInfo = MutableLiveData<DeviceInfo>()
    val deviceInfo: LiveData<DeviceInfo> get() = _deviceInfo

    fun updateDeviceInfo(newInfo: DeviceInfo) {
        _deviceInfo.postValue(newInfo) // 使用 postValue 确保线程安全
    }
}
```

#### 2. 在 SystemService 中更新 ViewModel
位置：`app/src/main/java/cn/xiaoa/app/smartdevice/SystemService.kt`

```kotlin
class SystemService : Service() {
    private lateinit var viewModel: DeviceInfoViewModel
    
    override fun onCreate() {
        super.onCreate()
        
        // 获取 ViewModel
        val app = application as SmartGlassesApp
        viewModel = app.deviceInfoViewModel
        
        // 初始化设备信息
        deviceInfo = DeviceInfo()
        viewModel.updateDeviceInfo(deviceInfo)
        
        // 当信息变更时更新 ViewModel
        deviceInfo.addOnChangeListener { newInfo ->
            viewModel.updateDeviceInfo(newInfo)
        }
    }
    
    // ... 其他代码
}
```

#### 3. 在 Application 类中管理 ViewModel
位置：`app/src/main/java/cn/xiaoa/app/smartdevice/SmartGlassesApp.kt`

```kotlin
class SmartGlassesApp : Application() {
    lateinit var deviceInfoViewModel: DeviceInfoViewModel
    
    override fun onCreate() {
        super.onCreate()
        deviceInfoViewModel = ViewModelProvider.AndroidViewModelFactory
            .getInstance(this)
            .create(DeviceInfoViewModel::class.java)
    }
}
```

#### 4. 在 MainActivity 中观察数据变化
位置：`app/src/main/java/cn/xiaoa/app/smartdevice/MainActivity.kt`

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: DeviceInfoViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 获取 ViewModel
        viewModel = (application as SmartGlassesApp).deviceInfoViewModel
        
        // 观察设备信息变化
        viewModel.deviceInfo.observe(this) { deviceInfo ->
            updateDeviceStatus(deviceInfo)
        }
    }
    
    private fun updateDeviceStatus(deviceInfo: DeviceInfo) {
        // 更新UI组件
        findViewById<TextView>(R.id.tvDeviceStatus).text = buildString {
            append("BLE: ${deviceInfo.bleStatus}\n")
            append("网络: ${deviceInfo.networkStatus}\n")
            append("LED亮度: ${deviceInfo.ledBrightness}%")
        }
        
        // 更新其他UI元素...
    }
}
```

### 备选方案：使用 EventBus（适用于简单场景）

#### 1. 添加 EventBus 依赖
`build.gradle (Module: app)`
```gradle
implementation 'org.greenrobot:eventbus:3.3.1'
```

#### 2. 创建事件类
位置：`app/src/main/java/cn/xiaoa/app/smartdevice/events/DeviceInfoUpdateEvent.kt`

```kotlin
package cn.xiaoa.app.smartdevice.events

import cn.xiaoa.app.smartdevice.model.DeviceInfo

class DeviceInfoUpdateEvent(val deviceInfo: DeviceInfo)
```

#### 3. 在 SystemService 中发送事件
```kotlin
class SystemService : Service() {
    // ...
    
    private fun onDeviceInfoChanged() {
        EventBus.getDefault().post(DeviceInfoUpdateEvent(deviceInfo))
    }
}
```

#### 4. 在 MainActivity 中接收事件
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onStart() {
        super.onStart()
        EventBus.getDefault().register(this)
    }
    
    override fun onStop() {
        super.onStop()
        EventBus.getDefault().unregister(this)
    }
    
    @Subscribe(threadMode = ThreadMode.MAIN)
    fun onDeviceInfoUpdate(event: DeviceInfoUpdateEvent) {
        updateDeviceStatus(event.deviceInfo)
    }
}
```

### 方案对比

| 特性                     | LiveData + ViewModel               | EventBus                  |
|--------------------------|------------------------------------|---------------------------|
| **生命周期感知**         | ✅ 自动管理                         | ❌ 需手动注册/注销         |
| **线程安全**             | ✅ 自动处理线程切换                 | ⚠️ 需指定线程模式           |
| **与 Android 架构集成**  | ✅ 官方推荐架构组件                 | ❌ 第三方库                 |
| **数据持久化**           | ✅ 配置变更时保留数据               | ❌ 数据不持久               |
| **代码复杂度**           | ⚠️ 中等（需理解架构组件）           | ✅ 简单易用                 |
| **实时性**               | ✅ 即时更新                         | ✅ 即时更新                 |
| **适用范围**             | 复杂应用、长期数据共享              | 简单应用、临时事件通知      |

### 最佳实践建议

1. **优先使用 LiveData + ViewModel**：
   - 这是 Android 官方推荐的架构模式
   - 自动处理生命周期和线程切换
   - 适合需要持久化数据的场景

2. **在 DeviceInfo 中添加变更监听器**：
   ```kotlin
   class DeviceInfo {
       // ... 现有属性
       
       private val listeners = mutableListOf<(DeviceInfo) -> Unit>()
       
       fun addOnChangeListener(listener: (DeviceInfo) -> Unit) {
           listeners.add(listener)
       }
       
       private fun notifyChanges() {
           listeners.forEach { it(this) }
       }
       
       // 在属性变更时调用 notifyChanges()
       var ledBrightness: Int = 80
           set(value) {
               field = value
               notifyChanges()
           }
   }
   ```

3. **使用 DiffUtil 优化 UI 更新**：
   如果 UI 更新频繁，可以在 RecyclerView 适配器中使用 DiffUtil 进行高效更新

4. **处理后台更新时的 UI 状态**：
   ```kotlin
   viewModel.deviceInfo.observe(this) { deviceInfo ->
       if (isVisible) { // 检查 Activity 是否在前台
           updateDeviceStatus(deviceInfo)
       } else {
           // 保存状态，在 onResume 时更新
       }
   }
   ```

### 完整流程说明

1. **数据变更流程**：
   ```
   SystemService 修改 DeviceInfo → 触发监听器 → 更新 ViewModel → LiveData 通知观察者 → Activity 更新 UI
   ```

2. **组件关系**：
   ```mermaid
   graph TD
   A[SystemService] -->|更新| B[DeviceInfo]
   B -->|通知| C[ViewModel]
   C -->|LiveData| D[MainActivity]
   D -->|显示| E[UI]
   ```

3. **线程处理**：
   - 后台服务使用 `postValue()` 确保线程安全
   - LiveData 自动将更新切换到主线程
   - 避免直接在后台线程更新 UI

这种架构确保了后台服务与 Activity 之间的高效、安全通信，同时符合现代 Android 开发的最佳实践。



