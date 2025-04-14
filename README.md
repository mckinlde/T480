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

## 📦 USB Installer Creation (macOS Monterey + OpenCore)

### Step 1: Download macOS Installer
- Used `macrecovery.py` from `OpenCorePkg/Utilities/macrecovery/`
- Ran from Python 3 on a working system (Windows, Linux, or macOS)
- Used this board ID (for `MacBookPro15,2`, which matches the T480’s CPU generation):
  ```bash
  python macrecovery.py download -b Mac-827FB448E656EC26 -os default
  ```
- Output location defaults to `./RecoveryImage/` with:
  - `BaseSystem.dmg` (✅ should be ~650 MB)
  - `BaseSystem.chunklist` (✅ small .plist-like file)

### Step 2: Prepare USB
- USB drive size: 8GB+ (used 128GB FAT32-formatted USB)
- Used **Rufus** with these settings:
  - Boot selection: **Non-bootable** (used to create FAT32 container)
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
- `AirportItlwm.kext` (v2.3.0 Monterey version only)

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

## ⬇️ macOS Monterey Install
1. Reboot → Press `F12` → Select OpenCore USB
2. At boot picker, select `EFI (external) (dmg)`
3. macOS Base System loads (verbose boot appears)
4. If pairing screen appears: connect USB keyboard/mouse
5. In Disk Utility:
   - View → Show All Devices
   - Select free space on internal SSD
   - Format as:
     - Name: `Macintosh HD`
     - Format: `APFS`
     - Scheme: `GUID Partition Map`
6. Close Disk Utility → Reinstall macOS Monterey
7. Select `Macintosh HD` as target

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
- Sequoia installer requires internet — Monterey doesn’t
- `BaseSystem.dmg` must be ~650MB or OpenCore will show “LoadImage failed - Unsupported”
- Ethernet confirmed working in recovery via `IntelMausi.kext`
- Russian-language UI may appear due to `prev-lang:kbd` → can be reset via NVRAM tool or `Reset NVRAM` from OpenCore
- Windows/NixOS entries may show in BIOS but not boot if EFI entries are broken — they can be restored later

---

✅ This flow will reliably install macOS Monterey to a Lenovo T480 and can be reused for additional machines with identical hardware.
