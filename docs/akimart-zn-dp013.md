# Akimart ZN-DP013 15.6" Digital Photo Frame (32GB Variant)

The Akimart ZN-DP013 is a 15.6-inch white-label display running Frameo on a Rockchip RK3326 chip. It allows standard ADB access through its USB-C port. 

This guide provides the steps to safely connect to the device, check its firmware properties, disable the stock launcher interface, and prevent background manufacturer updates.

## Prerequisites

1. **USB Cable:** A working USB-A to USB-C cable that supports data transfer.
2. **Android SDK Platform Tools**

---

## Step 1: Disconnecting from the Internet and Enabling ADB

To prevent the frame from downloading background updates before you can disable the stock interface and update services, keep the device disconnected from the internet during setup.

1. Turn on the frame. When prompted during the initial setup wizard, **skip the Wi-Fi setup**.
2. If your firmware requires a Wi-Fi connection to proceed, connect the frame to a local Wi-Fi network that has no internet access.
3. Open the **Settings** menu.
4. Go to **Join Beta Program**.
5. Toggle **ADB Access** to **ON**.

---

## Step 2: Setting Up Wireless ADB

You can configure ADB to run over Wi-Fi so you do not need to keep a USB cable connected.

1. Connect the frame to your computer via USB. Open the terminal on your computer and run these commands to set the persistent network properties:
   ```bash
   adb shell
   su
   setprop persist.adb.tcp.port 5555
   setprop service.adb.tcp.port 5555
   exit
   exit
   ```

2. Now restart the ADB daemon in network mode from your computer's terminal:
   ```bash
   adb tcpip 5555
   ```

3. Unplug the USB cable from your computer.

4. Connect the frame to your local Wi-Fi network. Find its IP address under **Settings** -> **Wi-Fi**.

5. Connect wirelessly from your computer's terminal:
   ```bash
   adb connect YOUR_FRAME_IP:5555
   ```

*Note: Using the persistent properties allows this network configuration to survive system reboots. You will only need to run the `adb connect` command again if the frame's IP address changes.*

---

## Step 3: Backing Up the Stock Frameo APK

Back up the original Frameo application to your computer before disabling any services.

1. Find the application path on the device:
   ```bash
   adb shell pm path net.frameo.frame
   ```
   
   *Expected Output:* `package:/system/priv-app/Frameo/Frameo.apk`

2. Pull the file to your computer (replace the path with the exact output from the above command):
   ```bash
   adb pull /system/priv-app/Frameo/Frameo.apk Frameo.apk
   ```
   
   *Note: The completed backup file size is approximately 64 MB (64,040,990 bytes).*

---

## Step 4: Checking Device Properties

Verify your device properties to confirm they match this hardware configuration.

1. Open the ADB shell:
   ```bash
   adb shell
   ```

2. Switch to the root user:
   ```bash
   su
   ```
   
   *The terminal prompt will change to # confirming root access.*

3. Run the following commands to check the firmware properties:
   ```bash
   getprop ro.board.platform
   getprop ro.build.version.release
   getprop ro.build.fingerprint
   getprop ro.build.display.id
   dumpsys package com.google.android.webview | grep versionName
   ```

### Expected Verified Log Targets
* **Platform:** rk3326
* **Android Version:** 8.1.0
* **Build Fingerprint:** rockchip/rk3326_go/rk3326_go:8.1.0/OC/084913:user/release-keys
* **Firmware Display ID:** SSA_Frame_20260108
* **WebView Engine Version:** Blank / Returns Error Status

---

## Step 5: Disabling the Stock Interface and Update Services

> [!WARNING]
> **SECURITY WARNING:** Enabling ADB Access grants root privileges over the network via port 5555 without authentication. Deploy this device only on an isolated network.

**CRITICAL WARNINGS:** 
1. Before disabling the stock photo interface, ensure you have already installed an alternative launcher or kiosk application (such as Nova Launcher, Fully Kiosk, or WallPanel) using `adb install path/to/app.apk`. If you disable the stock UI without an alternative launcher installed, the screen will go blank.
2. Ensure you have already completed **Step 3** to secure a backup of the stock APK before proceeding.

Run these commands inside the root shell (`adb shell` followed by `su`):

1. Identify the update services on the device:
   ```bash
   pm list packages | grep -E "update|ota|fota|card"
   ```
   
   *Expected Output:*
   ```text
   package:com.adups.fota
   package:android.rockchip.update.service
   package:com.android.dreams.phototable
   ```

2. Disable the automatic update services:
   ```bash
   pm disable --user 0 com.adups.fota
   pm disable --user 0 android.rockchip.update.service
   ```

3. Disable the primary stock Frameo user interface:
   ```bash
   pm disable --user 0 net.frameo.frame
   ```
   
   *The stock interface will stop running, and the device will switch to your custom launcher.*

---

## Step 6: Clearing Cache and Disconnecting

Run these final storage optimization and cleanup commands inside your root shell.

1. Do **NOT** run destructive directory wipes like `rm -rf /cache/*`, as this firmware may require the existing recovery structures inside `/cache`.

2. Clear out the residual application database and trim the filesystem:
   ```bash
   pm clear net.frameo.frame
   vdc fstrim dotrim
   ```
   
   *Note: Running `pm clear` will permanently wipe the stock Frameo user settings and photos. If you want to retain your stock configuration data as a fallback, you can safely skip the `pm clear` command.*

3. Exit the root shell and disconnect your computer from the frame:
   ```bash
   exit
   exit
   adb disconnect
   ```
