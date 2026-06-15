# Intel NUC 8i7BEH (Bean Canyon) macOS Tahoe 26.5.1 Hackintosh

[English](README.md) | 简体中文

本项目基于 OpenCore 1.0.8 引导，完美支持在 **Intel NUC 8i7BEH** 迷你主机上运行 **macOS Tahoe (26.5.1)**。 

本配置参考并致敬了经典项目 [Nucintosh](https://github.com/zearp/Nucintosh)，并针对 macOS Tahoe (26.x) 系统移除了原生声卡驱动（`AppleHDA`）以及无线网卡需要通过 `itlwm + HeliPort` 连接等新特性进行了深度的适配与调优。

---

## 💻 硬件配置与支持状态

| 硬件组件 | 型号 | 驱动方式 | 状态 |
| :--- | :--- | :--- | :--- |
| **处理器 (CPU)** | Intel Core i7-8559U | 原生电源管理 (`CPUFriend.kext`) |  完美变频 |
| **显卡 (iGPU)** | Intel Iris Plus Graphics 655 | `WhateverGreen.kext` (ID: `0x3EA50004`) |  图形加速 |
| **有线网卡** | Intel i219-V Gigabit Ethernet | `IntelMausi.kext` |  正常 |
| **无线网卡** | Intel Wireless-AC 9560 | `itlwm.kext` + HeliPort 客户端 |  正常 |
| **蓝牙 (BT)** | Intel Wireless Bluetooth | `IntelBluetoothFirmware` + `BlueToolFixup` |  正常 |
| **声卡 (Audio)** | Realtek ALC235 | OCLP-Mod (补回 AppleHDA) + `AppleALC` (ID: 3) |  正常 |
| **读卡器** | RTS522A PCI Express Card Reader | `RealtekCardReader.kext` |  正常 |
| **雷电/Type-C** | 内建 Thunderbolt 3 | ACPI 补丁适配 |  正常 |

---

## 🛠️ macOS Tahoe 特殊配置说明 (必读)

从 macOS Tahoe (26.x) 开始，苹果删除了老旧 Mac 所使用的 `AppleHDA.kext` 驱动。为了使内置声卡与无线网卡驱动在 Tahoe 上完美运行，本项目在 `config.plist` 中做了以下关键的安全降级配置：

1. **禁用安全启动**：`SecureBootModel` 被设为 **`Disabled`**。
   * *原因：这是使用 OCLP-Mod 写入驱动（Root Patch）且能正常开机引导的前提。若开启 `x86legacy` 会导致重启直接崩溃进入恢复模式。*
2. **放宽系统沙盒与签名**：`boot-args` 中加入了 **`amfi=0x80`**。
   * *原因：运行未签名的网卡和声卡补丁驱动所必需。*
3. **绕过升级与硬件检测限制**：使用了 `RestrictEvents.kext` 并配置了启动参数 **`revpatch=sb`**。
   * *原因：在 `SecureBootModel` 禁用的状态下，伪装安全启动，使系统能正常接收官方小版本增量更新 (OTA) 推送。*

---

## 🚀 安装与配置指引

### 第一步：准备 EFI 与生成三码
1. 将本项目整个 `EFI` 文件夹拷贝至您的启动盘 EFI 分区中。
2. 使用 `GenSMBIOS` 或者是 `OCAuxiliaryTools` 生成适用于 **`iMac20,2`**（或 `Macmini8,1`）的机型三码（MLB, SystemSerialNumber, SystemUUID）。
3. 用文本编辑器或配置工具打开 [config.plist](file:///Volumes/OC/EFI/OC/config.plist)，将其填入 `PlatformInfo -> Generic` 下对应的位置。

### 第二步：安装 macOS Tahoe 26.5.1
1. 按照标准的黑苹果安装步骤通过 USB 启动盘安装系统。
2. 首次进入系统后，您会发现此时**没有声音**，且**无法直接连接 WiFi**，这是正常现象，请继续下一步。

### 第三步：运行 OCLP-Mod 补回声卡与蓝牙驱动 (关键)
为了带回被苹果删除的 `AppleHDA` 并使声卡正常工作：
1. 下载并打开 [OCLP-Mod (OpenCore Legacy Patcher Mod)](https://github.com/laobamac/OCLP-Mod) 软件。
2. 点击 **Post-Install Root Patch** 按钮。
3. 点击 **Start Root Patching**，并输入您的开机密码。软件会自动为您补回声卡及其他外设的系统层驱动。
4. **重启电脑**。在进入 OpenCore 引导选单时，**必须执行一次 Reset NVRAM**（以加载新的内核安全变量），随后正常进入系统。
5. 此时打开 **系统设置 -> 声音**，声卡已完美工作！

### 第四步：使用 HeliPort 连接 WiFi
由于 `itlwm.kext` 将无线网卡模拟为了以太网接口，需要客户端连接：
1. 下载最新版的 **[HeliPort 客户端](https://github.com/OpenIntelWireless/HeliPort/releases)** (DMG格式)。
2. 将 **HeliPort.app** 拖入系统的 **应用程序** 文件夹并打开。
3. 您将在屏幕右上角看到 WiFi 图标。点击它，选择您的 WiFi 并输入密码连接。
4. 建议在 HeliPort 菜单中勾选 **“Launch at Login”**（开机启动），使其实现开机自动连接。

---

## ⚙️ 个性化调优 (可选)

### 1. 关于本机中的 CPU 正确显示
当前配置中包含了 `revcpu=1` 和 `revcpuname=Intel\ Core\ i7-8559U` 参数。
- 它会让“关于本机”界面正确将您的处理器识别为 `Intel Core i7-8559U`（而非“未知”）。您可以根据实际的 CPU 型号自行在 `boot-args` 中修改 `revcpuname` 的值。

### 2. 去除啰嗦模式（纯净开机）
目前为了排错，默认启用了 `-v` 参数（代码跑马灯开机）。如果您希望享受白苹果开机画面：
- 编辑 `config.plist` 中的 `boot-args`，删去 **`-v`** 即可。

---

## 🤝 致谢
- [Acidanthera](https://github.com/acidanthera) 提供的 OpenCore 引导及各类核心 Kext。
