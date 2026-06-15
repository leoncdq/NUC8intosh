# Intel NUC 8i7BEH (Bean Canyon) macOS Tahoe 26.5.1 Hackintosh

This project is based on OpenCore 1.0.8 and perfectly supports running **macOS Tahoe (26.5.1)** on an **Intel NUC 8i7BEH** mini-PC.

This configuration references and pays homage to the classic project [Nucintosh](https://github.com/zearp/Nucintosh), and has been deeply adapted and optimized for macOS Tahoe (26.x) systems, removing the native sound card driver (`AppleHDA`) and requiring the wireless network card to connect via `itlwm + HeliPort`.

---

## 💻 Hardware Configuration and Support Status

| Hardware Components | Model | Driver Type | Status |

| :--- | :--- | :--- | :--- |

| **Processor (CPU)** | Intel Core i7-8559U | Native Power Management (`CPUFriend.kext`) | Perfect Frequency Variables |

| **Graphics Card (iGPU)** | Intel Iris Plus Graphics 655 | `WhateverGreen.kext` (ID: `0x3EA50004`) | Graphics Acceleration |

| **Wired Network Adapter** | Intel i219-V Gigabit Ethernet | `IntelMausi.kext` | Normal |

| **Wireless Network Adapter** | Intel Wireless-AC 9560 | `itlwm.kext` + HeliPort Client | Normal |

| **Bluetooth (BT)** | Intel Wireless Bluetooth | `IntelBluetoothFirmware` + `BlueToolFixup` | Normal |

| **Sound Card (Audio)** | Realtek ALC235 | OCLP-Mod (Restores AppleHDA) + `AppleALC` (ID: 3) | Normal |

| **Card Reader** | RTS522A PCI Express Card Reader | `RealtekCardReader.kext` | Normal |

| **Thunderbolt/Type-C** | Built-in Thunderbolt 3 | ACPI Patch Compatibility | Normal |

---

## 🛠️ macOS Tahoe Special Configuration Notes (Must Read)

Starting with macOS Tahoe (26.x), Apple removed the `AppleHDA.kext` driver used by older Macs. To ensure the built-in sound card and wireless network card drivers function perfectly on Tahoe, this project implemented the following critical security downgrade configurations in `config.plist`:

1. **Disable Secure Boot:** `SecureBootModel` is set to **Disabled**.

* *Reason:** This is a prerequisite for using OCLP-Mod to write the driver (Root Patch) and for normal booting. Enabling `x86legacy` will cause a crash upon reboot, entering recovery mode.

2. **Relax System Sandbox and Signing:** `amfi=0x80` has been added to `boot-args`.

* *Reason:** Required for running unsigned network card and sound card patch drivers.

3. **Bypass Upgrade and Hardware Detection Restrictions:** `RestrictEvents.kext` is used, and the boot parameter `revpatch=sb` is configured.

* *Reason: To fake a secure boot when `SecureBootModel` is disabled, allowing the system to receive official minor version incremental updates (OTA) push notifications.*

---

## 🚀 Installation and Configuration Guide

### Step 1: Preparing EFI and Generating the Model Code

1. Copy the entire `EFI` folder of this project to the EFI partition of your boot disk.

2. Use `GenSMBIOS` or `OCAuxiliaryTools` to generate the model code (MLB, SystemSerialNumber, SystemUUID) suitable for **`iMac20,2`** (or `Macmini8,1`).

3. Open [config.plist](file:///Volumes/OC/EFI/OC/config.plist) with a text editor or configuration tool, and fill in the corresponding locations under `PlatformInfo -> Generic`.

### Step Two: Install macOS Tahoe 26.5.1

1. Follow the standard Hackintosh installation steps to install the system via USB boot disk.

2. Upon first entering the system, you will find that **there is no sound** and **you cannot directly connect to WiFi**. This is normal; please continue to the next step.

### Step Three: Run OCLP-Mod to Restore Sound Card and Bluetooth Drivers (Crucial)

To restore the `AppleHDA` removed by Apple and make the sound card work properly:

1. Download and open the [OCLP-Mod (OpenCore Legacy Patcher Mod)](https://github.com/laobamac/OCLP-Mod) software.

2. Click the **Post-Install Root Patch** button.

3. Click **Start Root Patching** and enter your login password. The software will automatically restore the system-level drivers for your sound card and other peripherals.

4. **Restart your computer**. Upon entering the OpenCore boot menu, **a Reset NVRAM must be performed once** (to load new kernel security variables), after which the system will boot normally.

5. Now open **System Settings -> Sound**, and the sound card should be working perfectly!

### Step Four: Connecting to WiFi using HeliPort

Since `itlwm.kext` emulates the wireless network card as an Ethernet interface, a client connection is required:

1. Download the latest version of the **[HeliPort client](https://github.com/OpenIntelWireless/HeliPort/releases)** (DMG format).

2. Drag **HeliPort.app** into the system's **Applications** folder and open it.

3. You will see a WiFi icon in the upper right corner of the screen. Click it, select your WiFi network, and enter the password to connect.

4. It is recommended to check **"Launch at Login"** in the HeliPort menu to enable automatic connection at boot.

---

## ⚙️ Personalized Tuning (Optional)

### 1. Correct CPU Display in About This Machine

The current configuration includes the parameters `revcpu=1` and `revcpuname=Intel\Core\i7-8559U`.

- This ensures the About This Machine interface correctly identifies your processor as `Intel Core i7-8559U` (instead of "Unknown"). You can modify the value of `revcpuname` in `boot-args` according to your actual CPU model.

### 2. Remove Verbose Mode (Clean Boot)

Currently, for troubleshooting purposes, the `-v` parameter (code-based boot animation) is enabled by default. If you want to enjoy the white Apple logo boot screen:

- Edit `config.plist` and remove **`-v`** from `boot-args`.

---

## 🤝 Acknowledgements

- [Acidanthera](https://github.com/acidanthera) provided the OpenCore bootloader and various core Kexts.
