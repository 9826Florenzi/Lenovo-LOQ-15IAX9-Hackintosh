# Lenovo LOQ 15IAX9 (Type 83GS) - Hackintosh OpenCore EFI

[![OpenCore](https://img.shields.io/badge/OpenCore-1.0.x-blue.svg)](https://github.com/acidanthera/OpenCorePkg)
[![macOS Support](https://img.shields.io/badge/macOS-Sonoma%20%7C%20Sequoia-success.svg)](https://www.apple.com/macos/)
[![CPU](https://img.shields.io/badge/CPU-Intel%2012th%20Gen%20i5--12450HX-lightgrey.svg)](https://ark.intel.com/content/www/us/en/ark/products/232155/intel-core-i5-12450hx-processor-12m-cache-up-to-4-40-ghz.html)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

An OpenCore EFI repository for running macOS on the **Lenovo LOQ 15IAX9 (Machine Type: 83GS)** laptop with the Intel 12th Gen Alder Lake-HX platform.

---

## 💻 Hardware Specifications

| Component | Detailed Specification | Device ID / Model |
| :--- | :--- | :--- |
| **Model** | Lenovo LOQ 15IAX9 (Type 83GS) | Motherboard: `LENOVO LNVNB161216` |
| **Chipset** | Intel HM670 Express Chipset | Alder Lake-HX Platform |
| **BIOS Version** | NECN34WW (UEFI) | |
| **Processor (CPU)** | Intel® Core™ i5-12450HX (12th Gen) | 8 Cores (4P + 4E), 12 Threads, up to 4.40 GHz |
| **Memory (RAM)** | 12 GB DDR5 4800 MHz (Ramaxel) | `RMSB3400KB06IVF-4800` |
| **Integrated GPU (iGPU)** | Intel® UHD Graphics for 12th Gen Processors | Alder Lake (No native macOS support, disabled via `-wegnoigpu`) |
| **Dedicated GPU (dGPU)** | NVIDIA® GeForce RTX™ 3050 6GB Laptop GPU | `10DE:25EC` (Subsystem: `17AA:3D06`) |
| **Display** | 15.6" Full HD (1920x1080), 144Hz / IPS | Internal eDP (`CSW150F`) |
| **Audio Codec** | Realtek® ALC257 High Definition Audio | `10EC:0257` (AppleALC `layout-id: 100`) |
| **Wi-Fi** | Intel® Wi-Fi 6 AX200 160MHz | PCIe `8086:2723` |
| **Bluetooth** | Intel® Wireless Bluetooth® | USB `8087:0029` |
| **Ethernet** | Realtek® PCIe GbE Family Controller (RTL8168/8111) | PCIe `10EC:8168` |
| **Storage (SSD)** | Micron 2400 NVMe SSD 512GB (DRAM-less) | PCIe NVMe `1344:5413` |
| **Touchpad** | Synaptics / Generic I2C HID Touchpad | I2C Multi-touch |
| **Keyboard** | Standard PS/2 Keyboard with Lenovo Fn Brightness keys | PS/2 |

---

## 📊 Current Status & What Works

| Feature | Status | Notes |
| :--- | :---: | :--- |
| **UEFI Boot & OpenCore** | ✅ Working | OpenCore 1.0.x with GUI Picker & Boot Chime |
| **CPU Power Management** | ✅ Working | Alder Lake spoofed as Comet Lake (`Cpuid1Data`) + `SSDT-PLUG-ALT` |
| **Realtek ALC257 Audio** | ✅ Working | Internal speakers, 3.5mm combo jack, and mic (`alcid=100`) |
| **Intel Wi-Fi (AX200)** | ✅ Working | Works via `itlwm.kext` + [HeliPort](https://github.com/OpenIntelWireless/HeliPort) client app |
| **Intel Bluetooth** | ✅ Working | Handled via `IntelBluetoothFirmware` + `IntelBTPatcher` + `BlueToolFixup` |
| **Realtek GbE Ethernet** | ✅ Working | Full gigabit speed with `RealtekRTL8111.kext` |
| **I2C Touchpad** | ✅ Working | Multi-touch gestures via `VoodooI2C` + `VoodooI2CHID` (`-vi2c-force-polling`) |
| **PS/2 Keyboard & Brightness** | ✅ Working | `VoodooPS2Controller` + `BrightnessKeys` |
| **Lenovo Fn & Battery Features**| ✅ Working | Battery percentage, conservation mode supported via `YogaSMC` |
| **NVMe SSD** | ✅ Working | Micron NVMe supported with power management via `NVMeFix.kext` |
| **USB Ports** | ✅ Working | Mapped with `USBToolBox` / `UTBDefault` |
| **Apple Services** | ⚠️ Partial | App Store & iCloud work; AirDrop / Universal Control requires Apple native Wi-Fi |
| **Graphics Acceleration** | ⚠️ Experimental | Intel 12th Gen iGPU has no Apple drivers. RTX 3050 uses NullMoth experimental display driver |

---

## ⚙️ Recommended BIOS Settings

Restart your laptop and press **F2** (or Fn+F2) repeatedly to enter the UEFI BIOS Setup:

### Disable:
- [x] **Secure Boot**
- [x] **Fast Boot**
- [x] **Intel VMD Controller / Intel RST** *(Crucial: Change Storage Controller mode from VMD to AHCI; Windows may require safe mode AHCI driver switch)*
- [x] **Wake on LAN**

### Enable:
- [x] **UEFI Mode** (Disable Legacy / CSM)
- [x] **Intel Virtualization Technology (VT-x)**
- [x] **SATA / NVMe in AHCI Mode**

---

## 🛠️ OpenCore Details & ACPI Fixes

### ACPI Patches & SSDTs
* **`DSDT.aml`**: System ACPI tables base.
* **`SSDT-PLUG-ALT.aml`**: Power management injection for Intel 12th Gen Alder Lake P-cores & E-cores.
* **`SSDT-EC.aml`**: Embedded controller fake device for macOS.
* **`SSDT-RTCAWAC.aml`**: Resolves RTC / AWAC timer conflicts in modern Intel chipsets.
* **`SSDT-GPI0.aml`**: Enables GPIO pinning for I2C Touchpad interrupts.
* **`SSDT-USB-Reset.aml` & `SSDT-USBX.aml`**: USB controller reset and power delivery properties.
* **`SSDT-SBUS.aml` & `SSDT-MCHC.aml`**: Fixes SMBus and memory controller identification.
* **`SSDT-XOSI.aml`**: Routes OS calls to simulate Windows environment for ACPI functions.
* **ACPI Delete Table Patch**: Drops OEM SSDT1 containing buggy Lenovo `PMAX` device (`TableLength: 908`, `OemTableId: CB-01   `) to prevent ACPI boot panics.

### Essential Kexts
* **`Lilu.kext`**: Arbitrary kext and process patching engine.
* **`VirtualSMC.kext`**: Advanced Apple SMC emulator (`SMCBatteryManager`, `SMCLightSensor`, `SMCProcessor`, `SMCSuperIO`).
* **`AppleALC.kext`**: Native audio patching for ALC257 (`alcid=100`).
* **`WhateverGreen.kext`**: Video patching (`-wegnoigpu` to disable Alder Lake iGPU).
* **`VoodooI2C.kext` + `VoodooI2CHID.kext`**: Precision I2C touchpad multi-touch driver.
* **`VoodooPS2Controller.kext`**: Laptop keyboard and TrackPoint/Mouse driver.
* **`BrightnessKeys.kext`**: Automatic handling of Fn brightness adjustment keys.
* **`YogaSMC.kext`**: Lenovo-specific hardware controls, battery thresholds, and conservation modes.
* **`itlwm.kext`**: Intel Wi-Fi driver (connect via HeliPort status bar app).
* **`IntelBluetoothFirmware.kext` + `IntelBTPatcher.kext` + `BlueToolFixup.kext`**: Full Intel Bluetooth support.
* **`RealtekRTL8111.kext`**: Gigabit Ethernet driver.
* **`NVMeFix.kext`**: NVMe power management and autonomous power state transitions.
* **`RestrictEvents.kext`**: CPU name cosmetic fix (`revcpu=1`, `revcpuname=12th Gen Intel(R) Core(TM) i5-12450HX`) and system update capabilities.
* **`AMFIPass.kext`**: AMFI compatibility for root patches and 3rd-party kexts.

---

## 🚀 Installation & Setup Guide

### ⚠️ IMPORTANT: Generate Your Own SMBIOS (Do NOT skip!)
For privacy and to ensure Apple ID/iCloud functionality without account blacklisting, the SMBIOS values in this repository have been replaced with placeholders (`GENERATE_YOUR_OWN`).

Before booting this EFI:
1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
2. Run `GenSMBIOS.bat` (Windows) or `GenSMBIOS.command` (macOS).
3. Select option **1** to download MacSerial.
4. Select option **2** to select `EFI/OC/config.plist`.
5. Select option **3** and enter model: `MacBookPro16,2`
6. GenSMBIOS will automatically inject a unique:
   * **SystemSerialNumber**
   * **SystemUUID**
   * **MLB (Board Serial Number)**
7. Set **ROM** in `PlatformInfo -> Generic` to your laptop's Ethernet MAC address (without colons).

### 📶 Setting Up Wi-Fi:
Because this EFI uses `itlwm.kext`:
1. Download and install [HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases).
2. Launch HeliPort and look for the Wi-Fi icon on the macOS menu bar.
3. Select your network và nhập mật khẩu.
4. Thêm HeliPort vào **System Settings > General > Login Items** để tự khởi động cùng máy.

---

## 🇻🇳 Hướng Dẫn Dành Cho Người Dùng Việt Nam

1. **Sao lưu trước khi thử nghiệm**: Đảm bảo bạn đã sao lưu dữ liệu quan trọng và EFI gốc của Windows trước khi can thiệp.
2. **Cài đặt BIOS**:
   * Bắt buộc phải tắt **Intel VMD / RST** trong BIOS (chuyển sang chế độ **AHCI**), nếu không bộ cài macOS sẽ không nhận diện được ổ cứng NVMe.
   * Tắt **Secure Boot** và **Fast Boot**.
3. **Tạo Serial Number cá nhân**:
   * File `config.plist` trong repo này đã được xóa các mã định danh cá nhân (`GENERATE_YOUR_OWN`).
   * Sử dụng công cụ **GenSMBIOS** để sinh mã mới cho mẫu máy `MacBookPro16,2` rồi điền vào `PlatformInfo > Generic`.
4. **Kết nối mạng Wi-Fi**:
   * EFI sử dụng `itlwm.kext` dành cho card Intel Wi-Fi AX200.
   * Sau khi vào màn hình macOS, tải ứng dụng **HeliPort**, kéo vào thư mục `Applications` và khởi chạy để bắt sóng Wi-Fi.

---

## 👏 Credits & Acknowledgements

* [Apple](https://www.apple.com/) for macOS.
* [Acidanthera](https://github.com/acidanthera) for OpenCore, Lilu, VirtualSMC, AppleALC, WhateverGreen, and RestrictEvents.
* [OpenIntelWireless](https://github.com/OpenIntelWireless) for `itlwm`, `HeliPort`, and `IntelBluetoothFirmware`.
* [alexandred](https://github.com/alexandred) and [VoodooI2C team](https://github.com/VoodooI2C/VoodooI2C) for the I2C trackpad drivers.
* [Zhen-Chao](https://github.com/zhen-chao) for `YogaSMC`.
* [Dortania](https://dortania.github.io/OpenCore-Install-Guide/) for the comprehensive OpenCore guides.
* [NullMoth team](https://github.com) for experimental display drivers.
