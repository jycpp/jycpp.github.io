---
title: WinPE安装Windows11的Security Violation错误
date: 2025-09-14 13:20:00
comments: true
tags:
- WinPE
- Windows11
- Security Violation
- Secure Boot
- Ventoy
- U盘启动
- BIOS
- TPM 2.0
- 系统安装
- 故障排除
categories:
- 兴趣爱好
- 系统维护
---


我们使用 Ventoy 制作的 U 盘启动盘，但是安装 Windows 11 时，出现了 Security Violation 错误，具体的错误一般为蓝屏状态下显示 `Verification failed: (0x1A) Security Violation`。

出现 `Verification failed: (0x1A) Security Violation` 错误，是因为一些较新的主板以及笔记本电脑的 Secure Boot（安全启动） 功能检测到你的 U 盘启动盘（特别是使用 Ventoy、微PE 等第三方工具制作的）没有经过微软或硬件厂商的数字签名认证，因此阻止了启动。这种现象在系统之家、电脑城、老毛桃等Win PE启动盘中特别常见。

针对这种错误，有以下两种解决方案，推荐优先使用方法一，因为它最简单且不需要修改 BIOS 设置。

方法一：在报错界面直接注册密钥（无需进 BIOS，最快）

如果你使用的是 Ventoy 或部分支持 Secure Boot 的 PE 工具，可以通过以下步骤一次性解决，以后重启不再报错：

1.  确认选项：当屏幕出现红色或黑色的 `Verification failed: (0x1A) Security Violation` 提示时，默认选中 “OK”，直接按 回车键（Enter）。
2.  进入管理菜单：屏幕会跳转到 `Perform MOK management` 界面，使用方向键选择 “Enroll key from disk”，按回车。
3.  选择磁盘：在 `Select Key` 界面，选择 “VTOYEFI”（如果是 Ventoy）或类似包含你 U 盘品牌/工具名称的选项，按回车。
    *   *注：如果是微PE或其他工具，可能显示为 `SHIM.EFI` 或具体文件名，选择对应的即可。*
4.  选择证书文件：找到并选择 `ENROLL_THIS_KEY_IN_MOKMANAGER.cer`（或类似 `.cer` / `.key` 结尾的文件），按回车。
5.  确认注册：
    *   在 `[Enroll MOK]` 界面，选择 “Continue”，按回车。
    *   在 `Enroll the key(s)?` 界面，选择 “Yes”，按回车。
6.  重启电脑：
    *   回到主菜单后，选择 “Reboot”，按回车。
    *   电脑重启后，再次从 U 盘启动，就不会再出现该报错，可以直接进入 PE 系统。

> 注意：此操作只需在第一次使用该 U 盘启动时执行一次，之后该 U 盘在这台电脑上均可正常启动。


![Security Violation Full.jpg](https://files.seeusercontent.com/2026/05/04/Kve5/Security-Violation-Full.jpg)

---

方法二：在 BIOS 中关闭 Secure Boot（通用方法）

如果方法一无效（例如某些旧版 PE 不支持密钥注册），或者你希望彻底关闭安全限制，可以进入 BIOS 关闭安全启动。

1.  进入 BIOS：
    *   关机状态下，按下电源键开机，并立即连续快速敲击 `F2` 键，直到进入 BIOS 设置界面。（ 有的电脑可能是 `F12` 或 `F11` 等其他键）
2.  找到安全启动选项：
    *   使用方向键切换到 “Security”（安全）或 “Boot”（启动）选项卡。
    *   找到 “Secure Boot” 选项，其状态当前应为 “Enabled”（已启用）。
3.  关闭安全启动：
    *   选中 “Secure Boot”，按回车，将其修改为 “Disabled”（禁用）。
    *   *注：部分笔记本电脑可能需要先设置 “Administrator Password”（管理员密码）才能修改 Secure Boot，若无法修改，请先设置一个简单密码（如 1234），修改完后再清除密码。*
4.  保存并退出：
    *   按 `F10` 键，选择 “Yes” 保存并退出。
5.  重新启动：
    *   电脑重启后，再次按 `F12` 选择 U 盘启动，即可直接进入 PE 系统，不再报错。

---

⚠️ 重要提示

1.  安装完成后建议重新开启 Secure Boot：
    *   Windows 11 官方要求开启 Secure Boot 以获得最佳安全性。系统安装完毕并进入桌面后，建议重新进入 BIOS 将 Secure Boot 改回 “Enabled”，否则可能导致部分游戏反作弊程序或银行插件无法运行。
2.  关于 TPM 2.0：
    *   关闭 Secure Boot 不会 影响 TPM 2.0 的状态。有一些品牌笔记本电脑 硬件支持 TPM 2.0，因此即使关闭了 Secure Boot，依然可以正常安装和激活 Windows 11。
3.  如果仍然无法启动：
    *   尝试更换 USB 接口（优先使用 USB 2.0 接口，若无则尝试另一个 USB 3.0 接口）。
    *   确保 U 盘制作时选择了 UEFI 模式（而非 Legacy/CSM），因为 Windows 11 必须使用 UEFI 引导。

按照上述任一方法操作后，你就可以顺利进入 PE 环境进行 Windows 11 的安装了。



