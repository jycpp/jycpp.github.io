---
title: Android Wi-Fi 连接控制技术演进与实现方案
date: 2025-06-21 09:15:30
comments: true
tags:
- Android
- Wi-Fi连接
- 权限管理
- WifiNetworkSuggestion
- WifiNetworkSpecifier
- Android 10
- Android 12
- 隐私保护
- 网络连接控制
- API演进
categories:
- 软件开发
- 运维管理
---



随着 Android 系统的不断发展，Wi-Fi 连接控制的实现方式也在不断变化。特别是从 Android 10 开始，Google 加强了对用户隐私和安全的保护，使得 Wi-Fi 连接的实现变得更加复杂。本文将详细介绍不同 Android 版本下 Wi-Fi 连接的实现方式及其技术要点。

## Android 10 之前版本的 Wi-Fi 连接

在 Android 10（API 29）之前，应用可以直接控制 Wi-Fi 连接，开发者可以通过简单的 API 调用来连接指定 SSID 和密码的 Wi-Fi 热点：

```kotlin
// Android 10以下连接方法
private fun connectWifiLegacy(wifiManager: WifiManager, ssid: String, password: String) {
    val wifiConfig = WifiConfiguration()
    wifiConfig.SSID = String.format("\"%s\"", ssid)
    wifiConfig.preSharedKey = String.format("\"%s\"", password)

    val netId = wifiManager.addNetwork(wifiConfig)
    if (netId != -1) {
        wifiManager.enableNetwork(netId, true)
        Log.d("Wi-Fi", "Network added with ID: $netId")
    } else {
        Log.e("Wi-Fi", "Failed to add network configuration")
    }
}
```


这种方式简单直接，应用可以完全控制 Wi-Fi 的连接和断开。

## Android 10 及以上版本的 Wi-Fi 连接

Android 10 成为了 Wi-Fi 连接控制的重要分水岭。从这个版本开始，Google 引入了新的权限模型和 API，使得应用无法直接强制连接 Wi-Fi 网络。

### 必要权限

在 Android 10 及以上版本中，连接 Wi-Fi 前需要请求以下四个基本权限：

1. `ACCESS_WIFI_STATE` - 访问 Wi-Fi 状态信息
2. `CHANGE_WIFI_STATE` - 修改 Wi-Fi 状态
3. `ACCESS_NETWORK_STATE` - 访问网络状态信息
4. `CHANGE_NETWORK_STATE` - 修改网络状态

### 连接方案

Android 10 提供了两种 Wi-Fi 连接方案：

#### 1. WifiNetworkSuggestion（建议连接）

`WifiNetworkSuggestion` 是一种推荐机制，应用向系统提供网络建议，由系统决定何时连接：

```kotlin
private fun connectWifiWithSuggestion(wifiManager: WifiManager, ssid: String, password: String) {
    val suggestion = WifiNetworkSuggestion.Builder()
        .setSsid(ssid)
        .setWpa2Passphrase(password)
        .build()

    val suggestionsList = listOf(suggestion)
    val status = wifiManager.addNetworkSuggestions(suggestionsList)
    
    when (status) {
        WifiManager.STATUS_NETWORK_SUGGESTIONS_SUCCESS -> {
            Log.d("Wi-Fi", "Successfully added network suggestion for $ssid")
        }
        // 处理其他状态
    }
}
```


**优点：**
- 符合 Android 的隐私保护设计理念
- 用户体验更好，用户有更多控制权
- 系统可以根据网络质量和信号强度智能选择

**缺点：**
- 无法保证立即连接
- 连接时机由系统决定
- 可能需要用户手动确认

#### 2. WifiNetworkSpecifier（强制连接）

`WifiNetworkSpecifier` 允许应用强制连接到指定网络：

```kotlin
private fun connectWifiWithSpecifier(wifiManager: WifiManager, ssid: String, password: String) {
    val specifier = WifiNetworkSpecifier.Builder()
        .setSsid(ssid)
        .setWpa2Passphrase(password)
        .build()

    val request = NetworkRequest.Builder()
        .addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
        .setNetworkSpecifier(specifier)
        .build()

    val networkCallback = object : ConnectivityManager.NetworkCallback() {
        override fun onAvailable(network: Network) {
            // 连接成功
        }

        override fun onUnavailable() {
            // 连接失败
        }
    }

    val connectivityManager = getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
    connectivityManager.requestNetwork(request, networkCallback)
}
```


**优点：**
- 可以强制连接到指定网络
- 连接控制更精确
- 不依赖系统决策

**缺点：**
- 可能断开当前网络连接
- 用户体验可能较差，因为连接之前会提示用户确认连接的Wifi热点名称，连接过程中还会提示用户连接进度
- 在某些版本中可能需要额外权限

## Android 12+ 的额外要求

从 Android 12（API 31）开始，连接 Wi-Fi 还需要额外申请 `ACCESS_FINE_LOCATION` 权限，因为 Wi-Fi 信息可以用于定位。这个权限需要在运行时请求：

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
    if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) 
        != PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions(this, 
            arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), REQUEST_CODE)
    }
}
```

以上过程中合计需要申请的权限在AndroidManifest.xml中的配置脚本如下：

```xml
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```


## 特殊情况下的 WRITE_SETTINGS 权限

在个别 Android 系统或定制 ROM 中，可能还需要 `android.permission.WRITE_SETTINGS` 权限，用于保存连接 Wi-Fi 的 SSID 和密码。这是一个特殊权限，需要用户在设置中手动授权：

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    if (!Settings.System.canWrite(this)) {
        val intent = Intent(Settings.ACTION_MANAGE_WRITE_SETTINGS)
        intent.data = Uri.parse("package:$packageName")
        startActivity(intent)
    }
}
```


## 总结

Android Wi-Fi 连接控制的演进体现了 Google 对用户隐私和安全保护的重视。开发者需要根据目标 Android 版本选择合适的连接方案，并正确处理权限申请。在实际开发中，建议：

1. 优先使用 `WifiNetworkSuggestion` 方案，符合现代 Android 设计理念
2. 仅在必要时使用 `WifiNetworkSpecifier` 强制连接
3. 正确处理各版本的权限要求
4. 提供良好的用户体验，避免强制切换网络影响用户正常使用

