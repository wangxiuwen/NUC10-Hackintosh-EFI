# Intel NUC 10 Hackintosh EFI - macOS Sonoma

This repository contains a fine-tuned OpenCore EFI for the **Intel NUC 10 (Frost Canyon)**, specifically optimized for **macOS Sonoma**.

## 💻 Hardware Configuration

| Component | specification | Notes / Details |
| :--- | :--- | :--- |
| **CPU** | Intel Core i7-10710U / i5-10210U | NUC10 specific |
| **GPU** | Intel UHD Graphics 620 | Natively supported via WhateverGreen |
| **Wi-Fi** | Intel Wi-Fi 6 AX201 | Works via `itlwm` / `AirportItlwm` |
| **Bluetooth** | Intel Bluetooth | Works via `IntelBluetoothFirmware` (See notes below) |
| **SD Card** | Genesys Logic GL9755 | Works via `GenericCardReaderFriend` / `AppleSDXC` |

## 🚀 Working Features

- CPU Power Management & Sleep/Wake functions.
- Intel UHD Graphics Hardware Acceleration.
- Realtek LAN Ethernet.
- Intel Wi-Fi & basic Bluetooth capabilities.
- Front and Rear Audio Jacks (ALC256).
- USB Ports (Mapped manually).
- **Integrated SD Card Reader (Genesys Logic GL9755)**.
- AirDrop & Handoff (Partially, reliant on Intel network stack spoofing).

## ⚠️ Known Limitations & Issues

### 1. Bluetooth Low Energy (BLE) audio dropout on Sonoma
Due to Apple's strict modifications to the Bluetooth protocol stack in macOS Monterey/Sonoma, the reverse-engineered `IntelBluetoothFirmware` kext **cannot handle BLE High-Fidelity Audio handshakes**.
- **Impact:** AirPods or BLE-based headphones will connect and then drop/disconnect immediately.
- **Workaround 1:** Use an older protocol or standard 2.4Ghz wireless headphones.
- **Workaround 2 (Ultimate Fix):** Replace the internal Intel AX201 card with a native Apple **BCM94360NG** M.2 Wi-Fi/Bluetooth card for 100% native support.

### 2. SD Card Reader Hardware
NUC10 uses the Genesys Logic GL9755 SD Card Reader (unlike NUC8 which used Realtek).
This EFI uses `GenericCardReaderFriend.kext` to spoof the reader to Apple's native `AppleSDXC` driver. It supports hot-plugging, but make sure not to have an SD card inserted *during boot*, as it may delay or stall the boot process occasionally.

## 🛠 Installation Tutorial

### Step 1: Prepare the USB Drive & BIOS
1. Create a macOS Sonoma offline installer USB using `createinstallmedia`.
2. Format the USB's EFI partition to FAT32.
3. Replace the `EFI` folder in your USB's EFI partition with the `EFI` folder from this repository.
4. **BIOS Settings for NUC10:**
    - Update BIOS to the latest version.
    - Disable Secure Boot.
    - Disable VT-d (Or keep enabled, OpenCore ignores it via `DisableIoMapper`).
    - Disable Fast Boot.
    - Set Boot Mode to UEFI ONLY.

### Step 2: Install macOS
1. Boot from the USB drive and select `Install macOS`.
2. Open Disk Utility, format your target SSD as **APFS (GUID Partition Map)**.
3. Install macOS to the SSD. It will reboot multiple times during this process. Make sure it boots from the SSD partition (not the external Installer) after the first reboot.

### Step 3: Post-Installation
1. Once fully booted into macOS, download [MountEFI](https://github.com/corpnewt/MountEFI) or use Terminal to mount the SSD's hidden EFI partition.
2. Copy the `EFI` folder from the USB drive to the SSD's EFI partition.
3. **Generate your own SMBIOS:**
   - Use [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) to generate new serial numbers for `Macmini8,1`.
   - Open `/EFI/OC/config.plist` using ProperTree or OpenCore Configurator.
   - Go to `PlatformInfo -> Generic` and paste your newly generated `SystemProductName`, `SystemSerialNumber`, `SystemUUID`, and `MLB`.
   - Save the config file.
4. Reboot! Your NUC10 is now a fully functional Hackintosh.

---

*This EFI is provided "as is". Always backup your data and existing EFI before testing new configurations.*
