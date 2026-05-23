---
title: "Installing a New Android Version on Phone: Ulefone Note 11P GSI Upgrade (Android 11 to Android 13)"
layout: post
---

In this guide, we document the complete process, commands, links, troubleshooting steps, and physical actions taken on a mobile device to upgrade the **Ulefone Note 11P** from stock Android 11 to **LineageOS 20 (Android 13) Generic System Image (GSI)**.

This tutorial is ideal if you want to breathe new life into your phone by installing a newer Android version.

---

## 📱 Device Specifications & Baseline
* **Device Model:** Ulefone Note 11P (Board: `g2062upt_v1_gd_sh2_gq_r`)
* **Chipset:** MediaTek Helio P60 (MT6771)
* **Stock OS version:** Android 11 (`GQ3095RH1_G2026U_Ulefone_20210616_V02`)
* **Partition Layout:** Dynamic Partitions (Project Treble Enabled) with slot suffix `_a` (A-only system within a logical slot container).

---

## 🛠️ Prerequisites & Setup

### 1. Platform-Tools
You must have the official Android SDK Platform-Tools (which contains `adb` and `fastboot`) downloaded on your workstation.
* **Mac Download Link:** [SDK Platform-Tools for Mac](https://developer.android.com/tools/releases/platform-tools)
* **Windows Download Link:** [SDK Platform-Tools for Windows](https://developer.android.com/tools/releases/platform-tools)
* **Linux Download Link:** [SDK Platform-Tools for Linux](https://developer.android.com/tools/releases/platform-tools)
* Extract the zip into your working directory (e.g., `./platform-tools`).

### 2. Physical Setup
* A reliable USB data cable.
* An external USB-A hub or multi-port USB adapter (essential workaround for Apple Silicon Macs).

---

## 📦 Required Files & Download Links

### 1. LineageOS 20 GSI
We use Andy Yan's TrebleDroid-based LineageOS 20 GSI. For a standard Google services experience, the `bgN` variant (includes GApps, no built-in root) is recommended.
* **GSI Downloads Directory:** [Andy Yan's SourceForge GSI Files](https://sourceforge.net/projects/andyyan-gsi/files/lineage-20-td/)
* **Direct File Link (bgN - GApps version):** [lineage-20.0-20251021-UNOFFICIAL-arm64_bgN-signed.img.gz](https://downloads.sourceforge.net/project/andyyan-gsi/lineage-20-td/lineage-20.0-20251021-UNOFFICIAL-arm64_bgN-signed.img.gz)
* **Extraction:** Uncompress the archive using `gunzip`:
  ```bash
  gunzip lineage-20.0-20251021-UNOFFICIAL-arm64_bgN-signed.img.gz
  ```
  This yields a raw system image (`lineage-20.0-20251021-UNOFFICIAL-arm64_bgN-signed.img`) of approximately 2.6 GB.

### 2. Custom Disabled VBMeta Image
To disable Android Verified Boot (AVB/dm-verity), we generate a custom `vbmeta_disabled.img` file.
1. Download the official AOSP `avbtool.py` script:
   * **Source Link:** [LineageOS Github Mirror of avbtool.py](https://raw.githubusercontent.com/LineageOS/android_external_avb/master/avbtool.py)
2. Generate the image using Python 3:
   ```bash
   python3 avbtool.py make_vbmeta_image --flags 2 --padding_size 4096 --output vbmeta_disabled.img
   ```
   This will output a 4096-byte `vbmeta_disabled.img` file in your working directory.

---

## 🔄 Step-by-Step Flashing Procedure

### Step 1: Enable USB Debugging on the Phone
1. On the phone, navigate to **Settings > About phone**.
2. Scroll to the bottom and tap **Build number** 7 times until you see the toast message: *"You are now a developer!"*.
3. Go back to **Settings > System > Developer options**.
4. Enable **USB debugging**.
5. Connect the phone to the computer via a USB cable.
6. When prompted on the phone screen with *"Allow USB debugging?"*, check **"Always allow from this computer"** and tap **Allow**.

#### 🔍 Troubleshooting USB Debugging:
* **Conflict with Google Chrome:** If ADB fails with an endpoint error (`ADB interface missing endpoints`), Google Chrome may have claimed the USB interface via WebUSB. Quit Chrome completely and replug the USB cable to resolve.
* **USB Mode:** If ADB still does not detect the device, swipe down on the phone screen, tap the USB charging notification, and change the USB mode from *No Data Transfer* to **PTP (Picture Transfer Protocol)** or **MTP**.

---

### Step 2: Enable "OEM Unlocking" (Device Action Required)
Android security blocks bootloader unlocking unless explicitly permitted in Developer Options.
1. Go to **Settings > System > Developer options**.
2. Find the **OEM unlocking** toggle and turn it **ON**.
3. Enter your PIN/pattern and tap **Enable** to confirm.

#### 🔍 Troubleshooting Missing / Greyed-out OEM Unlocking:
On many MediaTek devices (including Ulefone), the OEM Unlocking toggle is hidden by default until a check-in with Google's servers is successful.
* **Wi-Fi Connection:** Connect the device to a working Wi-Fi network.
* **Force Server Check-in:** Open the Phone/Dialer app and enter:
  `*#*#2432546#*#*` (spells `*#*#CHECKIN#*#*`)
  A notification saying *"check-in succeeded"* should appear. Restart the phone, and the toggle will be accessible.
* **Account Sync:** Ensure you are signed in to a primary Google account on the device.

---

### Step 3: Reboot to Bootloader & Unlock Bootloader
1. Open your terminal in the `platform-tools` folder.
2. Run the command to reboot the device:
   ```bash
   ./adb reboot bootloader
   ```
3. The phone screen will shut down and boot into a black screen with tiny white text showing "=> FASTBOOT mode" or show a boot menu.
4. Verify the device is detected:
   ```bash
   ./fastboot devices
   ```

#### 🔍 Troubleshooting Apple Silicon (M1/M2/M3) Fastboot Connection:
* **Issue:** On Apple Silicon Macs, `fastboot devices` will often return empty due to a handshake bug with the MediaTek fastboot interface over direct USB-C.
* **Workaround:** Plug the USB cable into an **external USB-A hub or adapter** instead of connecting it directly to the Mac's built-in Thunderbolt/USB-C ports. Replug the USB cable to force renegotiation.

5. Initiate the bootloader unlock:
   ```bash
   ./fastboot flashing unlock
   ```
6. **IMMEDIATE ACTION REQUIRED ON DEVICE:** Watch the phone screen. A warning page will appear asking you to confirm the bootloader unlock. **Press the Volume Up key immediately** (within 15 seconds) to select **Yes/Confirm**.
7. Confirm that the bootloader status says `unlocked: yes` by running:
   ```bash
   ./fastboot getvar unlocked
   ```

---

### Step 4: Flash VBMeta to Disable Verification
To prevent bootloops caused by system partition modifications, you must flash the generated disabled `vbmeta` image. We flash all three AVB metadata partitions (`vbmeta`, `vbmeta_system`, and `vbmeta_vendor`) to ensure no checks bypass:
```bash
./fastboot flash vbmeta vbmeta_disabled.img
./fastboot flash vbmeta_system vbmeta_disabled.img
./fastboot flash vbmeta_vendor vbmeta_disabled.img
```
*Note: Do not use the `--disable-verity` or `--disable-verification` flags in fastboot when flashing `vbmeta_disabled.img`, as these flags expect a signed factory partition image to patch, whereas we are flashing a pre-generated disabled structure.*

---

### Step 5: Boot into Fastbootd (User-space Fastboot)
The Ulefone Note 11P uses dynamic logical partitions inside the `super` partition container. You cannot flash the system partition in standard bootloader mode; you must boot into the Recovery-based `fastbootd` interface.
1. Run the command:
   ```bash
   ./fastboot reboot fastboot
   ```
2. The phone will reboot, and the screen will show **"FASTBOOTD"** (with options for Reboot system, Enter recovery, etc.).
3. **USB Replug Workaround (Apple Silicon):** Since the device rebooted into a new USB state, Apple Silicon Macs may lose connection. **Unplug and replug the USB cable** from the phone.
4. Verify that the device is back online:
   ```bash
   ./fastboot devices
   ```
5. Confirm that the mode is userspace:
   ```bash
   ./fastboot getvar is-userspace
   ```
   *(It must return `is-userspace: yes`)*

---

### Step 6: Flash the GSI System Image
1. Start the flashing command, pointing to the uncompressed `.img` file:
   ```bash
   ./fastboot flash system lineage-20.0-20251021-UNOFFICIAL-arm64_bgN-signed.img
   ```
2. The fastboot utility will automatically resize the logical `system_a` partition and upload the system image in sparse chunks (typically 11 chunks of ~262 MB).
3. Wait for the command to finish with `Finished. Total time: XXs` and return `OKAY`.

---

### Step 7: Wipe User Data (Factory Reset)
To prevent bootlooping due to old Android 11 database structures conflicting with Android 13, you must wipe all user data:
```bash
./fastboot -w
```
This command will erase the `userdata` and `metadata` partitions. 
*(If fastboot warns `Formatting is not supported for file system with type 'raw'`, this is normal. The android recovery system will automatically format these partitions on first boot).*

---

### Step 8: Boot into LineageOS 20
Reboot the device to boot into the new OS:
```bash
./fastboot reboot
```

---

## ⚡ First Boot Expectations
* **Unlocked Bootloader Warning:** You will see a warning message on boot screen stating: *"The bootloader is unlocked..."*. This is normal; it will auto-dismiss after 5 seconds.
* **First Boot Duration:** The initial boot can take between **2 to 5 minutes** while the phone formats the user space partitions and compiles core files.
* **Welcome Screen:** Once booted, you will be greeted by the **LineageOS Welcome Setup Wizard** where you can configure language, Wi-Fi, and Google services.

---

## 📶 Network Troubleshooting (No Internet / Wi-Fi & Cellular)

If the device boots successfully but is unable to connect to the internet via Wi-Fi or cellular data (or says "Connected, no internet"), follow these steps:

### 1. Disable Restricted Networking Mode
On some GSI builds, a security policy blocks user applications from accessing network interfaces.
1. Connect the phone to the computer via USB and ensure USB debugging is authorized.
2. Verify the restricted networking mode state:
   ```bash
   ./adb shell settings get global restricted_networking_mode
   ```
3. If it returns `1`, disable it by running:
   ```bash
   ./adb shell settings put global restricted_networking_mode 0
   ```
4. Reboot the phone:
   ```bash
   ./adb reboot
   ```

### 2. MediaTek / Phh Treble Settings Workarounds
Generic System Images (GSIs) require specific hardware overrides to communicate properly with MediaTek devices.
1. On the phone, navigate to **Settings > Phh Treble Settings** (or open the Phh Treble App).
2. Go to **Misc features** or **Tethering** configuration.
3. If mobile hotspot/tethering fails to share internet connection, enable **"Remove eBPF for tethering"** (which sets `persist.sys.phh.tethering.nobpf` to `1`) and reboot the device.
