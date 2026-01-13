# Qualcomm Android Monitor Mode (Snapdragon 8 Gen 2 / kiwi_v2)

This guide explains how to enable native monitor mode on Android devices with Qualcomm chipsets using the `kiwi_v2` driver (found in Snapdragon 8 Gen 2 and similar).

## Prerequisites
* **Root Access** (via Magisk or KernelSU).
* **ADB** installed on your PC.
* **iw binary**: A static or Android-compatible `iw` binary (aarch64). You can obtain this from Termux (`pkg install iw`) or specialized repositories.
* **tcpdump**: Usually pre-installed on many root environments or available via Termux.

## Installation & Setup

1. **Get the `iw` binary (via Termux):**
   Open Termux on your device and run:
   ```bash
   pkg install root-repo
   pkg install sudo iw
   sudo cp $PREFIX/bin/iw /sdcard/iw
   ```

2. **Move to an executable path and set permissions:**
   Connect via ADB and run:
   ```bash
   adb shell
   su
   cp /sdcard/iw /data/local/tmp/iw
   chmod 755 /data/local/tmp/iw
   ```

## Enabling Monitor Mode

Run the following commands as **root**:

```bash
# 1. Disable Android WiFi services to prevent interference
svc wifi disable
stop wpa_supplicant
pkill wpa_supplicant

# 2. Put the driver into monitor mode (con_mode 4)
ip link set wlan0 down
echo 4 > /sys/module/kiwi_v2/parameters/con_mode
ip link set wlan0 up

# 3. Set your desired frequency/channel (e.g., Channel 1 - 2412MHz)
/data/local/tmp/iw dev wlan0 set freq 2412

# 4. Verify the interface state
/data/local/tmp/iw dev wlan0 info
```

## Capturing Traffic

You can now use `tcpdump` to capture raw 802.11 frames:

```bash
# Live capture on terminal
tcpdump -i wlan0 -e

# Save to a .pcap file for Wireshark analysis
tcpdump -i wlan0 -w /sdcard/capture.pcap
```

## Restoration (Back to Managed Mode)

To restore normal WiFi functionality:

```bash
ip link set wlan0 down
echo 0 > /sys/module/kiwi_v2/parameters/con_mode
ip link set wlan0 up
svc wifi enable
start wpa_supplicant
```

---
*Note: This method was tested on the Snapdragon 8 Gen 2 (kalama) platform.*
