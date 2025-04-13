# OpenCore Triple-Boot Setup on Lenovo T480

This document tracks all configuration steps taken to set up a triple-boot system (Windows 11 + NixOS + macOS Monterey) on a Lenovo T480 using OpenCore.

---

## ✅ Summary of Hardware
- Model: Lenovo ThinkPad T480 (Intel-only)
- CPU: Intel Core i5-8350U (8th Gen, Kaby Lake-R)
- GPU: Intel UHD 620 (iGPU only, no NVIDIA)
- Ethernet: Intel I219-LM
- Wi-Fi: Intel Dual Band Wireless-AC 8265
- RAM: 32 GB DDR4 @ 2400 MHz

---

## 📦 USB Installer Creation (macOS + OpenCore)

### Step 1: Download macOS Installer
- Used `macrecovery.py` (from OpenCore guide) to download macOS Monterey (version 12)
- Output files:
  - `BaseSystem.dmg`
  - `BaseSystem.chunklist`

### Step 2: Format USB (Single GPT + FAT32 partition)
- Used **Rufus**:
  - Boot selection: Non bootable
  - Partition scheme: GPT
  - File system: FAT32
  - Cluster size: 32 KB (default)

### Step 3: Copy macOS Recovery Files
- Created directory on USB: `com.apple.recovery.boot`
- Copied files:
  - `BaseSystem.dmg`
  - `BaseSystem.chunklist`

### Step 4: Add OpenCore EFI
- Downloaded latest OpenCore release (v1.0.4)
- Used `Sample.plist` from `Docs/Sample.plist` as base config
- Renamed to `config.plist`, copied to `EFI/OC`
- Copied full `EFI` folder to root of USB

Final USB layout:
```
USB\
├── com.apple.recovery.boot\
│   ├── BaseSystem.dmg
│   └── BaseSystem.chunklist
└── EFI\
    ├── BOOT\
    └── OC\
        └── config.plist
```

---

## 🧩 Kexts Added (macOS Monterey-compatible)
Downloaded and extracted to: `C:\Users\dougl\Downloads\Kext`
Then copied only the `.kext` folders (not the release folders) to: `E:\EFI\OC\Kexts`

### Required Kexts:
| Kext | Purpose |
|------|---------|
| Lilu.kext | Core patching engine |
| VirtualSMC.kext | Emulates Apple SMC (optional plugins like SMCBatteryManager.kext not included for now) |
| WhateverGreen.kext | iGPU graphics, brightness, etc. |
| AppleALC.kext | Audio support |
| IntelMausi.kext | Ethernet support (I219-LM) |
| AirportItlwm.kext (v2.3.0 Monterey) | Wi-Fi support for Intel 8265 |

---

## 🔧 Tools Used
- [ProperTree](https://github.com/corpnewt/ProperTree) — for editing `config.plist`
- [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) — for generating valid SMBIOS values
- [Rufus](https://rufus.ie) — for formatting USB and SD card
- Windows `diskpart` — for manual partitioning and disk cleanup
- [macrecovery.py](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/windows-install.html) — for downloading BaseSystem from Apple
- [GParted Live](https://gparted.org/livecd.php) — to shrink NixOS partition safely

---

## ✅ Configuring OpenCore (Pre-Boot Setup)

### Step 1: Clean Snapshot in ProperTree
- Opened `config.plist` using ProperTree.bat
- Ran **Clean Snapshot** (`Ctrl+Shift+R`) against `E:\EFI\OC\`
- Verified that all kexts appeared in `Kernel → Add`

### Step 2: Generate SMBIOS
- Ran `GenSMBIOS.bat`
- Installed MacSerial (option 1)
- Selected config.plist (option 2 → `E:\EFI\OC\config.plist`)
- Generated SMBIOS for model: `MacBookPro15,2`
- Verified entries in `PlatformInfo → Generic`:
  - `SystemProductName`: `MacBookPro15,2`
  - `SystemSerialNumber`, `MLB`, `SystemUUID`, `ROM` (base64) all populated
  - `SpoofVendor`: `true`
  - `UpdateSMBIOS`, `UpdateNVRAM`, `UpdateDataHub`: `true`

### Step 3: Set NVRAM Boot Args
- Updated `boot-args` to:
  ```
  -v keepsyms=1 debug=0x100
  ```
- Verified in: `NVRAM → Add → 7C436110-AB2A-4BBB-A880-FE41995C9F82`

### Step 4: Set Booter → Quirks
| Quirk                    | Value  |
|--------------------------|--------|
| AvoidRuntimeDefrag       | true   |
| DevirtualiseMmio         | true   |
| EnableSafeModeSlide      | true   |
| ProtectMemoryRegions     | true   |
| ProtectSecureBoot        | false  |
| RebuildAppleMemoryMap    | true   |
| SetupVirtualMap          | true   |
| SignalAppleOS            | true   |
| SyncRuntimePermissions   | true   |

### Step 5: Set Kernel → Quirks
| Quirk                      | Value  |
|----------------------------|--------|
| AppleCpuPmCfgLock          | true   |
| AppleXcpmCfgLock           | true   |
| DisableIoMapper            | true   |
| DisableLinkeditJettison    | true   |
| EnableKernelPm             | true   |
| LapicKernelPanic           | false  |
| PowerTimeoutKernelPanic    | true   |
| ThirdPartyDrives           | false  |
| XhciPortLimit              | false  |

### Step 6: UEFI Drivers Check
Confirmed presence and enabled state of:
- `OpenRuntime.efi` ✅
- `OpenHfsPlus.efi` ✅
- `OpenCanopy.efi` ✅

Other non-essential drivers/tools were cleaned up or disabled for clarity and faster boot.

---

## ⏳ To Do Before Boot
- [x] Configure OpenCore and kexts
- [ ] Shrink NixOS partition to leave **50–100GB unallocated space**
  - Created GParted Live bootable SD card using Rufus
  - Boot into GParted Live from SD card using `F12`
  - Identify NixOS partition (`ext4` or `btrfs`)
  - Right-click → **Resize/Move** → shrink from the right
  - Leave desired unallocated space (do not format)
  - Apply changes and reboot

---

## ⬇️ macOS Installer Instructions
Once the USB is ready:
1. Reboot and press `F12` to open BIOS boot menu
2. Select the USB drive (should show up as UEFI)
3. At OpenCore boot picker, choose the **macOS Installer**

### In macOS Recovery
1. Open **Disk Utility**
   - View → Show All Devices
   - Select the unallocated space → erase as:
     - Name: `Macintosh HD`
     - Format: `APFS`
     - Scheme: `GUID Partition Map`
2. Quit Disk Utility
3. Launch **macOS Installer** and select `Macintosh HD`

---

## 🧹 Immediate Post-Install Steps (After First Boot)
- Boot again from USB
- Select newly installed `macOS` (not installer)
- Once in desktop:
  - Mount EFI of internal disk
  - Copy your working `EFI` folder from USB to internal disk EFI partition
  - Use [MountEFI](https://github.com/corpnewt/MountEFI) or `diskutil` to do this on macOS

After reboot, you can remove the USB stick.

---

Continue adding to this file as we proceed through macOS post-install (USB mapping, kext updates, etc.).
