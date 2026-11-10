# OpenCore EFI for Lenovo ThinkPad X395

[![OpenCore](https://img.shields.io/badge/OpenCore-1.0.x-blue.svg)](https://github.com/acidanthera/OpenCorePkg)
[![macOS](https://img.shields.io/badge/macOS-Tahoe%2026%20%7C%20Sequoia%2015-success.svg)](https://www.apple.com/macos/)
[![Hardware](https://img.shields.io/badge/Hardware-AMD%20Ryzen%20Picasso-orange.svg)](https://www.amd.com/)

A fully tuned, plug-and-play OpenCore EFI configuration for the **Lenovo ThinkPad X395** running **macOS Tahoe (macOS 26 / Darwin 25)** and **macOS Sequoia (macOS 15)** with full AMD Radeon Vega 8 graphics acceleration.

---

## 💻 Hardware Specifications

| Component | Detail | Status |
| :--- | :--- | :---: |
| **Model** | Lenovo ThinkPad X395 (Type 20NL, 20NM) | - |
| **Processor** | AMD Ryzen 5 PRO 3500U (4 Cores / 8 Threads, 2.1 - 3.7 GHz, Picasso) | ✅ Working |
| **Graphics** | AMD Radeon Vega 8 Graphics (Raven/Picasso GFX9) - 2048 MB VRAM | ✅ Working (Metal) |
| **Display** | 13.3" FHD (1920x1080) IPS Anti-glare | ✅ Working |
| **Audio** | Realtek ALC257 / ALC3287 (AppleALC `alcid=86` + SimpleHDA 1.0.1) | ✅ Working |
| **Wi-Fi** | Intel Wireless-AC 9260 (AirportItlwm for macOS Tahoe) | ✅ Working (Native UI) |
| **Bluetooth** | Intel Wireless Bluetooth (IntelBluetoothFirmware + IntelBTPatcher 2.5.0) | ✅ Working |
| **Trackpad** | Synaptics / ELAN PS/2 ClickPad with Multi-touch gestures | ✅ Working |
| **TrackPoint**| ThinkPad TrackPoint (red pointing stick + 3 physical buttons) | ✅ Working |
| **Storage** | M.2 NVMe PCIe SSD | ✅ Working |
| **Battery** | Internal Li-ion Battery with SMBus status readout | ✅ Working |
| **USB** | USB 3.1 Gen 2 Type-C & Type-A (Custom UTBMap) | ✅ Working |
| **SMBIOS** | `MacBookPro16,2` | Configured |

---

## ✨ Features & What's Working

- [x] **Full Graphics Acceleration:** AMD Radeon Vega 8 Graphics accelerated via `NootedRed.kext` with Metal 2/3 and Quartz Extreme.
- [x] **Internal & External Audio:** Working internal speakers, microphone, and 3.5mm combo jack via `SimpleHDA.kext` (which restores `AppleHDA` on macOS Tahoe) and `AppleALC.kext` (`layout-id=86`).
- [x] **Native Wi-Fi:** Intel Wireless-AC 9260 connects natively with macOS menu bar UI via `AirportItlwm.kext` (Tahoe alpha build).
- [x] **Bluetooth:** Full scanning, pairing, and peripheral connections (headphones, mice, keyboards) using `IntelBluetoothFirmware` and `IntelBTPatcher.kext` (v2.5.0).
- [x] **Trackpad & TrackPoint:** Smooth scrolling, pinch-to-zoom, tap-to-click, and physical button controls tuned via `VoodooPS2Controller.kext`.
- [x] **Display Brightness & Volume Keys:** Brightness slider and Fn shortcuts work natively (`BrightnessKeys.kext` & `SSDT-PNLF`).
- [x] **Power Management & Sleep/Wake:** Working battery status and ACPI sleep/wake routines.
- [x] **Direct Fast Boot:** Configured for instant direct boot into macOS without showing the boot picker (`ShowPicker = False`).

---

## ⚙️ Recommended BIOS Settings

Restart your ThinkPad X395 and press `F1` or `Enter` to access BIOS Setup:

### Required:
* **Config $\rightarrow$ Display:**
  * **Total Graphics Memory:** Set to **256MB**, **512MB**, or **2GB (UMA Buffer)** (*Recommended: 2GB / 2048MB for optimal graphics performance*).
* **Security $\rightarrow$ Security Chip:**
  * **Security Chip:** Disabled (or can be left Enabled if needed).
* **Security $\rightarrow$ Secure Boot:**
  * **Secure Boot:** **Disabled** (Mandatory).
* **Startup:**
  * **UEFI/Legacy Boot:** **UEFI Only**.
  * **CSM Support:** **No**.

---

## 🚀 Installation Guide

### Step 1: Prepare macOS Installer
1. Create a bootable macOS installer on a USB flash drive (16 GB+) using Apple's official `createinstallmedia` tool or via OpenCore's guide.
2. Mount the EFI partition of your USB drive (using tools like MountEFI, OpenCore Configurator, or Terminal `diskutil`).

### Step 2: Copy EFI
1. Clone or download this repository.
2. Copy the `EFI` folder to the root of your USB drive's EFI partition:
   ```text
   EFI (Partition)
   └── EFI
       ├── BOOT
       │   └── BOOTx64.efi
       └── OC
           ├── ACPI
           ├── Drivers
           ├── Kexts
           ├── Resources
           └── config.plist
   ```

### Step 3: Generate SMBIOS (Important!)
Before booting, generate your own unique serial numbers to avoid Apple ID conflicts:
1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
2. Run GenSMBIOS and select `EFI/OC/config.plist`.
3. Choose `MacBookPro16,2` as the SMBIOS model.
4. Save the configuration.

### Step 4: Boot and Install
1. Insert the USB installer into your ThinkPad X395 and power on while tapping `F12` to enter the boot menu.
2. Select your USB drive.
3. If the OpenCore boot picker does not show up (due to `ShowPicker=False`), hold the **`Escape`** or **`Spacebar`** key right after selecting the USB drive to bring up the picker.
4. Choose **Reset NVRAM** once.
5. Boot into the **Install macOS** option and proceed with installation.
6. Once installation is complete, copy the `EFI` folder from the USB drive to the internal SSD's EFI partition.

---

## 💡 Post-Installation Notes & Known Fixes

### 1. Chromium & Electron Apps (Google Chrome, Spotify, Discord, VS Code)
Because Apple only designed Vega drivers for discrete GPUs (e.g. iMac Pro), Chromium's ANGLE engine triggers a command processor stall on AMD APUs with shared UMA memory when GPU rasterization is active.

**To prevent system freeze when using Chromium apps:**

* **Google Chrome:**
  Run this command in Terminal once to permanently disable hardware acceleration:
  ```bash
  sudo defaults write /Library/Preferences/com.google.Chrome HardwareAccelerationModeEnabled -bool false
  ```
* **Spotify:**
  1. Open Spotify.
  2. In the top macOS menu bar, click **Help** $\rightarrow$ **Troubleshooting**.
  3. Click **Disable Hardware Acceleration and Restart**.
* **Visual Studio Code / Discord / Slack:**
  Launch once with `--disable-gpu` or disable Hardware Acceleration in the app's internal preferences.
* **Alternative Browsers:**
  **Safari** and **Mozilla Firefox** work natively with full GPU hardware acceleration without any freezing!

### 2. Restoring the Boot Menu Picker
If you want the OpenCore boot menu to show up on every boot:
* In `EFI/OC/config.plist`, go to `Misc` $\rightarrow$ `Boot`:
  * Set `ShowPicker` to `True`.
  * Set `Timeout` to `5`.

### 3. Bluetooth Reset (If Scanning Loops)
If Bluetooth gets stuck in a scanning loop after an improper shutdown:
```bash
sudo nvram bluetoothExternalDongleFailed=%00
sudo rm -rf /Library/Preferences/com.apple.Bluetooth.plist
sudo pkill -9 bluetoothd
```

---

## 👏 Credits & Acknowledgements

- [Apple](https://www.apple.com) for macOS.
- [Acidanthera](https://github.com/acidanthera) for OpenCore, Lilu, AppleALC, VirtualSMC, VoodooPS2, and BrcmPatchRAM.
- [ChefKissInc](https://github.com/ChefKissInc/NootedRed) for NootedRed (AMD Vega iGPU acceleration).
- [OpenIntelWireless](https://github.com/OpenIntelWireless) for AirportItlwm, IntelBluetoothFirmware, and IntelBTPatcher.
- [laobamac](https://github.com/laobamac) for SimpleHDA (restoring AppleHDA on macOS Tahoe) and AirportItlwm Tahoe build.
- [Dortania](https://dortania.github.io/OpenCore-Install-Guide/) for the comprehensive OpenCore guides.
