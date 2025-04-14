# OpenCore Hackintosh Setup on Lenovo T480 (macOS Sequoia / Monterey)

This document captures the full, trial-tested process to set up a fully bootable macOS system on the Lenovo T480 using OpenCore. It has been adjusted through hard-earned experience and includes every detail necessary for reinstallation or replication.

---

## ✅ System Overview
- Model: Lenovo ThinkPad T480
- CPU: Intel Core i5-8350U (8th Gen)
- GPU: Intel UHD 620
- Wi-Fi: Intel 8265 (requires custom kext)
- Ethernet: Intel I219-LM
- Input: Built-in PS/2 keyboard + trackpad
- macOS: Sequoia (15.x), optionally Monterey (12.x)

---

## 📦 Creating the Bootable USB

### Step 1: Download macOS Recovery
Run on a working Mac or PC with Python 3 installed:
```bash
python macrecovery.py download -b Mac-827FB448E656EC26 -os default
```

Check that you get:
- `BaseSystem.dmg` (~650 MB)
- `BaseSystem.chunklist` (few KB)

Place both in:
```
USB/com.apple.recovery.boot/
```

### Step 2: Format USB (Rufus)
- Partition scheme: GPT
- File system: FAT32
- Set up USB root layout:
```
USB/
├── EFI/
│   ├── BOOT/
│   └── OC/
└── com.apple.recovery.boot/
```

### Step 3: Essential Kexts for USB
Place these in `EFI/OC/Kexts/`:
- `Lilu.kext`
- `VirtualSMC.kext`
- `WhateverGreen.kext`
- `AppleALC.kext`
- `IntelMausi.kext`
- `AirportItlwm.kext` (for Intel 8265 Wi-Fi)
- `VoodooPS2Controller.kext` (for built-in keyboard/trackpad)

🧠 Optional additions:
- `USBToolBox.kext` (USB mapping)
- `NVMeFix.kext` (SSD power fixes)
- `HeliPort.app` (menu bar UI for Wi-Fi if using `itlwm.kext`)

Snapshot the USB EFI with **ProperTree** (`Ctrl + Shift + R`) and save `config.plist`.

---

## 🧩 OpenCore Config Notes
- SMBIOS: `MacBookPro15,2`
- Booter and Kernel quirks: snapshot from validated USB config
- `ScanPolicy`: 0
- `Vault`: Optional
- `SecureBootModel`: Disabled

---

## 🔧 Internal Installation Process

### 1. Boot USB → Select `EFI (external) (dmg)`
### 2. In macOS Recovery:
- View → Show All Devices
- Format unallocated disk space as:
  - Name: `Macintosh HD`
  - Format: `APFS`
  - Scheme: `GUID Partition Map`

### 3. If Installing Sequoia
✅ **Plug in Ethernet BEFORE booting**  
Sequoia requires network validation even if your Wi-Fi isn’t working yet.

### 4. Complete macOS Install → Reboot
- Use OpenCore USB to continue installation
- On first boot, complete Setup Assistant (skip Apple ID for now)

---

## 🔁 Finalize Internal EFI
1. Mount internal EFI (likely `disk0s1`):
```bash
sudo diskutil mount disk0s1
```
2. Copy full `EFI/` folder from USB → `/Volumes/EFI/`
3. Add missing kexts (like `VoodooPS2Controller.kext`, `AirportItlwm.kext`)
4. Run ProperTree → Clean Snapshot → Save
5. Reboot and test

If booting fails without USB:
- Boot from USB → OpenShell → map EFI partition
- Run:
```bash
bcfg boot add 0 fs1:\EFI\BOOT\BOOTx64.efi "OpenCore Internal"
```
- `reset` → Boot without USB

---

## ✅ Working State
- Internal OpenCore boots macOS Sequoia
- Internal keyboard/trackpad functional
- Wi-Fi requires `AirportItlwm.kext` + snapshot

---

## 🧠 Lessons Learned
- Always verify kext presence in `/Kexts` before snapshotting
- Snapshot every time you add/remove a kext
- Use ProperTree from a working Mac if GUI fails on target system
- Sequoia forces network validation → Monterey may be easier for first-time installs
- Keep full-featured USB EFI for future use or cloning

---

## 📦 Next Steps
- Zip this working EFI folder for backup or replication
- Reinstall Windows or NixOS if desired
- Optional: create automated EFI image for use across multiple T480s

---

**This process is non-trivial and time-consuming—but fully documented here for reuse. Proceeding from a clean, working EFI is the best gift to yourself and others.**
