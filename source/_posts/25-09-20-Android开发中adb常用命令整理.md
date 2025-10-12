---
title: Android开发中adb常用命令整理
date: 2025-09-20 14:35:22
comments: true
tags:
- Android
- ADB
- 安卓Root
- 无线调试
categories:
- 软件开发
- 小程序和APP
---


这里为你整理了一份非常全面且分类清晰的 Android ADB 命令清单，涵盖了开发、调试和高级操作的各个方面。

---

### **ADB 简介**

**ADB (Android Debug Bridge)** 是一个命令行工具，它是 Android SDK 的一部分，用于与模拟器或实体设备进行通信。它是 Android 开发者的瑞士军刀。

---

## **一、基础与连接命令**

| 命令 | 描述 |
| :--- | :--- |
| `adb devices` | **列出所有连接的设备和模拟器**。这是最常用的命令，用于检查连接状态。 |
| `adb connect <device-ip>:5555` | 通过 **TCP/IP** 连接到设备（需设备开启网络调试）。 |
| `adb disconnect <device-ip>:5555` | 断开 TCP/IP 连接。 |
| `adb kill-server` | 终止 ADB 服务器进程。 |
| `adb start-server` | 启动 ADB 服务器进程。 |
| `adb reboot` | **重启设备**。 |
| `adb -s <device-serial> <command>` | **指定设备执行命令**。当连接多台设备时，用此命令指定目标设备。 |

---

## **二、应用安装与管理**

| 命令 | 描述 |
| :--- | :--- |
| `adb install path/to/app.apk` | **安装 APK**。 |
| `adb install -r path/to/app.apk` | **覆盖安装**（保留数据）。 |
| `adb install -t path/to/app.apk` | 允许安装测试包（用于 `android:testOnly="true"` 的应用）。 |
| `adb uninstall <package-name>` | **卸载应用**（例如 `com.example.app`）。 |
| `adb uninstall -k <package-name>` | 卸载应用但**保留数据和缓存目录**。 |
| `adb shell pm list packages` | **列出所有已安装应用的包名**。 |
| `adb shell pm list packages -3` | 列出所有**第三方安装**的应用包名。 |
| `adb shell pm path <package-name>` | 显示指定 APK 的安装路径。 |
| `adb shell pm clear <package-name>` | **清除指定应用的数据和缓存**。 |

---

## **三、文件操作**

| 命令 | 描述 |
| :--- | :--- |
| `adb push <local> <remote>` | **将电脑文件推送到设备**（如 `adb push file.txt /sdcard/`）。 |
| `adb pull <remote> <local>` | **将设备文件拉取到电脑**（如 `adb pull /sdcard/file.txt .`）。 |
| `adb shell ls <path>` | 列出设备上目录的内容。 |
| `adb shell cd <path>` | 切换目录（通常在 `adb shell` 内使用）。 |
| `adb shell rm <path>` | 删除文件。 |
| `adb shell rm -r <path>` | 递归删除目录。 |

---

## **四、日志与调试**

| 命令 | 描述 |
| :--- | :--- |
| `adb logcat` | **显示设备日志**（实时滚动）。 |
| `adb logcat -v time` | 显示日志，并包含时间戳。 |
| `adb logcat *:E` | **仅显示错误 (Error) 级别的日志**。 |
| `adb logcat -s <TAG>` | 仅显示指定标签 (TAG) 的日志（例如 `adb logcat -s MyApp`）。 |
| `adb logcat > log.txt` | **将日志输出保存到电脑的文件中**。 |
| `adb shell dumpsys <service>` | ** dump 系统服务信息**（非常强大）。 |
| `adb shell dumpsys activity activities` | 查看当前 Activity 栈信息。 |
| `adb shell dumpsys meminfo <package-name>` | 查看指定应用的内存使用情况。 |
| `adb shell dumpsys battery` | 查看电池信息。 |
| `adb shell dumpsys package <package-name>` | 查看应用的详细安装信息。 |
| `adb shell am start -n <package>/<activity>` | **启动一个 Activity**（例如 `adb shell am start -n com.example.app/.MainActivity`）。 |
| `adb shell am force-stop <package-name>` | **强制停止一个应用**。 |
| `adb shell am broadcast -a <intent>` | 发送一个广播 (Broadcast)。 |

---

## **五、设备信息与控制**

| 命令 | 描述 |
| :--- | :--- |
| `adb shell getprop` | **获取全部系统属性**。 |
| `adb shell getprop ro.product.model` | 获取设备型号。 |
| `adb shell getprop ro.build.version.release` | 获取 Android 系统版本。 |
| `adb shell wm size` | **显示屏幕分辨率**。 |
| `adb shell wm density` | 显示屏幕密度 (DPI)。 |
| `adb shell screencap -p /sdcard/screen.png` | **截取屏幕截图并保存到设备**。 |
| `adb shell screenrecord /sdcard/demo.mp4` | **录制屏幕**（默认 3 分钟，按 `Ctrl+C` 停止）。 |
| `adb shell input keyevent <keycode>` | **模拟按键输入**（如 `KEYCODE_HOME`=3, `KEYCODE_BACK`=4）。 |
| `adb shell input text "text"` | **模拟文本输入**。 |
| `adb shell input tap <x> <y>` | **模拟触摸点击**。 |
| `adb shell input swipe <x1> <y1> <x2> <y2>` | **模拟滑动**。 |

---

## **六、Root 相关命令（需设备已 Root）**

**重要提示：以下命令需要在已获得 Root 权限的设备上，并且通过 `adb root` 命令或 `su` 提升权限后才能执行。**

| 命令 | 描述 |
| :--- | :--- |
| `adb root` | **以 Root 权限重启 ADB 守护进程**。这是执行高级 Root 命令的前提。 |
| `adb shell whoami` | 检查当前 Shell 的用户身份，成功获取 Root 后会显示 `root`。 |
| `adb shell` | 进入普通的 Shell。 |
| `adb shell su -c "<command>"` | **在 Shell 中执行单条 Root 命令**（常用）。例如：`adb shell su -c "pm install /sdcard/app.apk"` |
| `adb remount` | 重新挂载 `/system` 分区为可读写（在 `adb root` 后使用）。 |
| `adb disable-verity` | 禁用分区验证（通常用于在开发版系统上执行 `adb remount`）。 |
| `adb shell pm install -r -d -t /sdcard/app.apk` | 以 Root 权限安装 APK（绕过某些限制）。 |
| `adb shell pm uninstall -k <package-name>` | 以 Root 权限卸载应用。 |
| `adb push local.file /system/app/` | **将文件推送到系统分区**（需先 `adb remount`）。 |
| `adb shell cat /proc/kmsg` | 查看内核日志 (Kernel Log)。 |
| `adb shell ls -l /data/data/<package-name>/` | **查看其他应用的数据目录**（需要 Root）。 |

---

## **七、无线调试（无需USB）**

1.  **让设备监听 TCP/IP 连接**（通常需要先用 USB 连接）：
    ```bash
    adb tcpip 5555
    ```
2.  **找到设备的 IP 地址**（设置 -> 关于手机 -> 状态信息）。
3.  **通过 IP 连接设备**：
    ```bash
    adb connect 192.168.1.100:5555
    ```
4.  **断开 USB 线缆**，现在可以无线调试了。
5.  **切换回 USB 模式**：
    ```bash
    adb usb
    ```

---

## **八、实用技巧与组合命令**

| 命令 | 描述 |
| :--- | :--- |
| `adb shell ps \| grep <package-name>` | 查找与应用相关的进程。 |
| `adb logcat \| grep -i "error"` | (在 Mac/Linux 上) 过滤日志中的错误。 |
| `adb logcat \| findstr "error"` | (在 Windows 上) 过滤日志中的错误。 |
| `adb shell am start -a android.settings.SETTINGS` | 打开系统设置界面。 |
| `adb shell cmd package resolve-activity -a android.intent.action.MAIN` | 查看 Launcher Activity。 |

希望这份清单能成为你开发过程中的得力助手！


