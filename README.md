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
| **Processor** | AMD Ryzen 5 PRO 3500U (4 Cores / 8 Threads, 2.1 - 3.7 GHz, Picasso) | ✅ Working (ForgedInvariant TSC Sync) |
| **Graphics** | AMD Radeon Vega 8 Graphics (Raven/Picasso GFX9) - 2048 MB VRAM | ✅ Working (Shaneee PAT + Metal 2/3) |
| **Display** | 13.3" FHD (1920x1080) IPS Anti-glare | ✅ Working |
| **Audio** | Realtek ALC257 / ALC3287 (AppleALC `alcid=86` + SimpleHDA 1.0.1) | ✅ Working |
| **Wi-Fi** | Intel Wireless-AC 9260 (AirportItlwm for macOS Tahoe) | ✅ Working (Native UI) |
| **Bluetooth** | Intel Wireless Bluetooth (IntelBluetoothFirmware + IntelBTPatcher 2.5.0) | ✅ Working |
| **Trackpad** | Synaptics / ELAN PS/2 ClickPad (VoodooPS2Controller 2.3.7 + `SkipPassThrough`) | ✅ Working |
| **TrackPoint**| ThinkPad TrackPoint (red pointing stick + 3 physical buttons) | ✅ Working |
| **Storage** | M.2 NVMe PCIe SSD (NVMeFix 1.1.3 + Native APFS TRIM) | ✅ Working (APST Enabled) |
| **Battery** | Internal Li-ion Battery with SMBus status readout | ✅ Working |
| **USB** | USB 3.1 Gen 2 Type-C & Type-A (Custom UTBMap) | ✅ Working |
| **SMBIOS** | `MacBookPro16,2` (ProcessorType `1537` / `0x0601`) | Configured |

---

## ✨ Features & Tuning Highlights

- [x] **Full Graphics Acceleration:** AMD Radeon Vega 8 Graphics accelerated via `NootedRed.kext` with Metal 2/3 and Quartz Extreme.
- [x] **Shaneee PAT Patch:** Uses the high-performance Shaneee Fix PAT patch (`_mtrr_update_action` Write-Combining `0x0606060606060606`) to prevent AMD GPU write-combining bottlenecks and TLB flush hangs.
- [x] **TSC Synchronization:** Equipped with `ForgedInvariant.kext` (v1.5.0) to synchronize the Time Stamp Counter (TSC) across AMD cores and prevent `mp_cpus_call_wait()` kernel panics.
- [x] **NVMe Power Management & Stability:** Equipped with `NVMeFix.kext` (v1.1.3) supporting APST (Autonomous Power State Transition), active thermal throttling prevention, and timeout fixes for OEM drives (like Samsung PM981/PM981a).
- [x] **Intel MKL Acceleration:** Includes `IntelMKLFixup.kext` (v1.0.0) allowing Intel Math Kernel Library apps (Adobe Photoshop CC, Lightroom CC, MATLAB, NumPy) to run natively without crashes on AMD.
- [x] **Native CPU Brand Display:** Tuned `PlatformInfo -> Generic -> ProcessorType = 1537` (`0x0601`) to correctly display `AMD Ryzen 5 PRO 3500U` in *About This Mac*.
- [x] **Robust Trackpad & Gestures:** Updated to `VoodooPS2Controller.kext` v2.3.7 with `SkipPassThrough = True` to prevent TrackPoint multiplexing desync and mid-session freezes.
- [x] **Internal & External Audio:** Working internal speakers, microphone, and 3.5mm combo jack via `SimpleHDA.kext` (restores `AppleHDA` on macOS Tahoe) and `AppleALC.kext` (`layout-id=86`).
- [x] **Native Wi-Fi & Bluetooth:** Intel Wireless-AC 9260 connects natively via `AirportItlwm.kext` and `IntelBluetoothFirmware` + `IntelBTPatcher.kext` (v2.5.0).
- [x] **Display Brightness & Volume Keys:** Brightness slider and Fn shortcuts work natively (`BrightnessKeys.kext` & `SSDT-PNLF`).
- [x] **Direct Fast Boot:** Configured for instant direct boot into macOS without showing the boot picker (`ShowPicker = False`).

---

## ⚙️ Recommended BIOS Settings

Restart your ThinkPad X395 and press `F1` or `Enter` to access BIOS Setup:

### Required:
* **Config $\rightarrow$ Power:**
  * **Sleep State:** Set to **`Linux`** (Mandatory for traditional S3 Suspend-to-RAM sleep; *Windows 10 / Modern Standby* causes sleep/wake issues on macOS).
  * **Wake on LAN:** **Disabled**.
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

## 💡 Post-Installation Notes & Essential Fixes

### 1. Mandatory Sleep & Power Management Settings (`pmset`)
To prevent darkwake black screen, hybrid sleep hangs, or RTC wake loops on AMD Ryzen APUs, configure macOS power management in Terminal:
```bash
sudo pmset -a hibernatemode 0 standby 0 autopoweroff 0 powernap 0 proximitywake 0 tcpkeepalive 0
sudo rm -rf /var/vm/sleepimage
```

### 2. Chromium & Electron Apps (Google Chrome, Spotify, Discord, VS Code)
Because Apple only designed Vega drivers for discrete GPUs, Chromium's ANGLE rasterization engine can cause GPU hangs on AMD APUs with shared UMA memory when hardware acceleration is active.

* **Google Chrome:**
  ```bash
  sudo defaults write /Library/Preferences/com.google.Chrome HardwareAccelerationModeEnabled -bool false
  ```
* **Discord:**
  Run in Terminal:
  ```bash
  killall Discord 2>/dev/null; python3 -c 'import json, os; dirs = ["discord", "discordcanary", "discordptb"]; [os.path.exists(os.path.dirname(p)) and (lambda d: (d.update({"enableHardwareAcceleration": False}), open(p, "w").write(json.dumps(d, indent=2))))(json.load(open(p)) if os.path.exists(p) else {}) for p in [os.path.expanduser(f"~/Library/Application Support/{d}/settings.json") for d in dirs]]'
  ```
* **Spotify:**
  1. Open Spotify.
  2. In the top macOS menu bar, click **Help** $\rightarrow$ **Troubleshooting**.
  3. Click **Disable Hardware Acceleration and Restart**.
* **Visual Studio Code:**
  Add `"disable-hardware-acceleration": true` in `argv.json` (accessible via `Preferences: Configure Runtime Arguments` in VS Code).
* **Native Browsers:**
  **Safari** and **Mozilla Firefox** run with full native GPU hardware acceleration without freezing!

### 3. NVMe Storage Performance (Samsung PM981a / Other NVMe)
* `NVMeFix.kext` (v1.1.3) is pre-configured to manage APST and keep operating temperatures low.
* Verify TRIM is enabled by running:
  ```bash
  system_profiler SPNVMeDataType | grep "TRIM Support:"
  ```
  *(Should output `TRIM Support: Yes`)*.

### 4. Restoring the Boot Menu Picker
If you want the OpenCore boot menu to show up on every boot:
* In `EFI/OC/config.plist`, go to `Misc` $\rightarrow$ `Boot`:
  * Set `ShowPicker` to `True`.
  * Set `Timeout` to `5`.

### 5. Bluetooth Reset (If Scanning Loops)
If Bluetooth gets stuck in a scanning loop after an improper shutdown:
```bash
sudo nvram bluetoothExternalDongleFailed=%00
sudo rm -rf /Library/Preferences/com.apple.Bluetooth.plist
sudo pkill -9 bluetoothd
```

---

## 👏 Credits & Acknowledgements

- [Apple](https://www.apple.com) for macOS.
- [Acidanthera](https://github.com/acidanthera) for OpenCore, Lilu, AppleALC, VirtualSMC, VoodooPS2, NVMeFix, and BrcmPatchRAM.
- [ChefKissInc](https://github.com/ChefKissInc/NootedRed) for NootedRed (AMD Vega iGPU acceleration) and ForgedInvariant (TSC sync).
- [Shaneee](https://github.com/Shaneee) for the AMD Shaneee Fix PAT patch.
- [OpenIntelWireless](https://github.com/OpenIntelWireless) for AirportItlwm, IntelBluetoothFirmware, and IntelBTPatcher.
- [laobamac](https://github.com/laobamac) for SimpleHDA (restoring AppleHDA on macOS Tahoe) and AirportItlwm Tahoe build.
- [Dortania](https://dortania.github.io/OpenCore-Install-Guide/) for the comprehensive OpenCore guides.
