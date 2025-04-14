# OpenCore Triple-Boot Setup on Lenovo T480

This document tracks all configuration steps taken to set up a triple-boot system (Windows 11 + NixOS + macOS Monterey or Sequoia) on a Lenovo T480 using OpenCore.

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
- Used `macrecovery.py` from `OpenCorePkg/Utilities/macrecovery/`
- Ran from Python 3 on a working system (Windows, Linux, or macOS)
- Used board ID for `MacBookPro15,2` (matches T480's hardware):
  ```bash
  python macrecovery.py download -b Mac-827FB448E656EC26 -os default
  ```
- Output location defaults to `./RecoveryImage/` with:
  - `BaseSystem.dmg` (~650–700 MB)
  - `BaseSystem.chunklist` (a few KB)

### Step 2: Prepare USB
- USB drive size: 8GB+ (used 128GB FAT32-formatted USB)
- Used **Rufus** with:
  - Boot selection: **Non-bootable**
  - Partition scheme: **GPT**
  - File system: **FAT32**, Cluster size: 32 KB (default)

### Step 3: Populate USB
- Created folder:
  ```
  USB\com.apple.recovery.boot\
  ```
- Copied `BaseSystem.dmg` and `BaseSystem.chunklist` into that folder
- Downloaded [OpenCorePkg RELEASE](https://github.com/acidanthera/OpenCorePkg/releases)
- Copied the `EFI` folder from OpenCorePkg to:
  ```
  USB\EFI\
  ```
- Created and edited config.plist based on `Docs/Sample.plist`
- Used **ProperTree** to snapshot and save configuration

Final USB layout:
```
USB\
├── com.apple.recovery.boot\
│   ├── BaseSystem.dmg
│   └── BaseSystem.chunklist
└── EFI\
    ├── BOOT\
    │   └── BOOTx64.efi
    └── OC\
        ├── OpenCore.efi
        ├── config.plist
        ├── Kexts\
        └── Drivers\
```

---

## 🧩 Kexts Included in EFI
Copied to `EFI\OC\Kexts`:
- `Lilu.kext`
- `VirtualSMC.kext`
- `WhateverGreen.kext`
- `AppleALC.kext`
- `IntelMausi.kext`
- `AirportItlwm.kext` (v2.3.0 Monterey version)

Post-install, also include:
- `VoodooPS2Controller.kext` (for built-in keyboard/trackpad)

---

## 🔧 config.plist Configuration Summary
- Used **MacBookPro15,2** SMBIOS
- Generated using `GenSMBIOS`
- Applied snapshot with **ProperTree**

### PlatformInfo → Generic:
- `SystemProductName`: `MacBookPro15,2`
- `ROM`: base64 MAC or generated
- `SystemUUID`: random UUID from GenSMBIOS

### Booter Quirks:
- AvoidRuntimeDefrag: `true`
- DevirtualiseMmio: `true`
- EnableSafeModeSlide: `true`
- ProtectMemoryRegions: `true`
- RebuildAppleMemoryMap: `true`
- SetupVirtualMap: `true`
- SignalAppleOS: `true`
- SyncRuntimePermissions: `true`

### Kernel Quirks:
- AppleCpuPmCfgLock: `true`
- AppleXcpmCfgLock: `true`
- DisableIoMapper: `true`
- DisableLinkeditJettison: `true`
- EnableKernelPm: `true`
- PowerTimeoutKernelPanic: `true`

### Misc → Security:
- ScanPolicy: `0`
- SecureBootModel: `Disabled`
- Vault: `Optional`

### UEFI → Drivers:
- Only include:
  - `OpenRuntime.efi`
  - `OpenHfsPlus.efi`
  - `OpenCanopy.efi`

---

## ⏳ Pre-Boot Setup
- Used GParted Live to shrink NixOS partition (~58 GB)
- Windows was previously shrunk (~25 GB)
- Left ~84 GB unallocated total for macOS

---

## ⬇️ macOS Installation Notes

### Monterey (Offline Install):
- Does not require internet
- Can proceed directly to Disk Utility + Install

### Sequoia (Online Install):
- Requires internet **at boot** to validate installer
- Ethernet must be connected **before powering on** the machine
- Verified working via `IntelMausi.kext`
- macOS will otherwise fail with "Internet connection required" popup

### Disk Utility:
1. View → Show All Devices
2. Select unallocated space or full disk
3. Format as:
   - Name: `Macintosh HD`
   - Format: `APFS`
   - Scheme: `GUID Partition Map`

Then proceed with install.

---

## 🧹 Post-Install Steps
- Boot back into OpenCore USB
- Select installed macOS
- After setup:
  - Mount EFI partition of internal SSD
  - Copy `EFI\` folder from USB to internal EFI
  - Optional: Add `VoodooPS2Controller.kext` for internal input
  - Snapshot updated config.plist again with ProperTree

---

## Notes & Gotchas
- Ethernet must be plugged in **before boot** to satisfy Sequoia installer
- macOS Sequoia will require network; Monterey does not
- `BaseSystem.dmg` must be ~650MB or OpenCore shows “LoadImage failed - Unsupported”
- If only Windows/NixOS entries appear in OpenCore but don't boot, EFI bootloaders were likely wiped and will need reinstalling
- Russian (or other language) UI may default if `prev-lang:kbd` is set in NVRAM — can be reset via OpenCore’s `Reset NVRAM`

---

✅ This process now supports installing **macOS Monterey or Sequoia** cleanly on a T480 and can be reused for additional machines with identical hardware.
