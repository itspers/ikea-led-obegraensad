# Flashing Guide - IKEA LED OBEGRÄNSAD

This guide documents how to flash firmware to the ESP32-S board, both via USB and wirelessly (OTA).

## Initial Setup (Done Once)

### What We Did

1. **Installed PlatformIO** via pipx:
   ```bash
   pipx install platformio
   ```

2. **Built and flashed firmware** for ESP32-S board:
   ```bash
   cd /Users/stas/Apps/home/ikea-led-obegraensad
   pio run -e esp32dev -t upload
   ```

3. **Board detected at**: `/dev/cu.usbserial-0001` (CP2102 USB to UART Bridge Controller)

---

## USB Flashing (When Connected via USB)

### Prerequisites

- PlatformIO installed (`pipx install platformio` if not already installed)
- ESP32-S board connected via USB
- Project directory: `/Users/stas/Apps/home/ikea-led-obegraensad`

### Steps

1. **Check connected device**:
   ```bash
   pio device list
   ```
   Look for your ESP32 board (usually `/dev/cu.usbserial-*` on macOS)

2. **Build the firmware**:
   ```bash
   cd /Users/stas/Apps/home/ikea-led-obegraensad
   pio run -e esp32dev
   ```
   (Use `esp32s3` environment if you have a Seeed Xiao ESP32-S3 board)

3. **Upload firmware**:
   ```bash
   pio run -e esp32dev -t upload
   ```

4. **Monitor serial output** (optional, to see device logs):
   ```bash
   pio device monitor
   ```
   Or use PlatformIO's serial monitor in VS Code.

### Environment Selection

- **ESP32 Dev Board**: `esp32dev` (default)
- **Seeed Xiao ESP32-S3**: `esp32s3`
- **Wemos D1 Mini ESP32**: `ESP32-wemos`

Check `platformio.ini` for all available environments.

---

## OTA (Over-The-Air) Updates (Wireless)

OTA updates allow you to upload firmware wirelessly without a USB connection. This is convenient once the device is installed in the lamp.

### Prerequisites

1. **Device must be connected to WiFi** (configured via WiFi Manager on first boot)
2. **Know the device IP address** (check router admin panel or serial monitor)
3. **OTA credentials configured** in `include/secrets.h`:
   ```cpp
   #define OTA_USERNAME "admin"
   #define OTA_PASSWORD "ikea-led-wall"
   ```

### Method 1: PlatformIO Automated Upload (Recommended)

1. **Edit `platformio.ini`** - Uncomment and configure OTA settings for your environment:

   For `esp32dev` environment (around line 72):
   ```ini
   [env:esp32dev]
   extends = env:esp32-base
   board = esp32dev
   extra_scripts = upload.py
   upload_protocol = custom
   custom_upload_url = http://192.168.1.XXX  # Replace with your device IP
   custom_username = admin
   custom_password = ikea-led-wall
   ```

2. **Build and upload**:
   ```bash
   pio run -e esp32dev -t upload
   ```
   PlatformIO will automatically upload via OTA instead of USB.

**Note**: Make sure the device IP matches what's configured in `custom_upload_url`.

### Method 2: Web Interface (Manual Upload)

1. **Build the firmware**:
   ```bash
   pio run -e esp32dev
   ```

2. **Find firmware file**:
   ```
   .pio/build/esp32dev/firmware.bin
   ```

3. **Open browser** and navigate to:
   ```
   http://YOUR-DEVICE-IP/update
   ```
   Example: `http://192.168.1.50/update`

4. **Login** with OTA credentials:
   - Username: `admin`
   - Password: `ikea-led-wall` (or whatever you set in `secrets.h`)

5. **Select firmware file** (`firmware.bin` from step 2) and click "Update"

6. **Wait for completion** - Device will reboot automatically

### Visual Feedback During OTA

- **"U" letter** displayed on LED matrix: Update started
- **Serial output**: Progress updates every second
- **"R" letter** displayed: Update completed (device reboots)

---

## WiFi Configuration

### First-Time Setup

The ESP32 uses **WiFiManager** - you don't need to hardcode WiFi credentials:

1. **After flashing**, device boots and tries to connect to known networks
2. **If no known network**, device creates WiFi AP named **`IKEA`** (configurable in `include/constants.h`)
3. **Connect** your phone/laptop to the **`IKEA`** network
4. **Captive portal** opens automatically (or navigate to `http://192.168.4.1`)
5. **Select your WiFi** network and enter password
6. **Device saves credentials** and reboots, then connects to your WiFi

### Changing Setup AP Name

Edit `include/constants.h`:
```cpp
#define WIFI_MANAGER_SSID "IKEA"  // Change to your preferred name
```

### Finding Device IP

- Check serial monitor output after boot
- Check your router's admin panel (look for hostname `ikea-led`)
- Use mDNS: `http://ikea-led.local` (if mDNS works on your network)

---

## Troubleshooting

### USB Flashing Issues

- **Port not found**: Check USB cable and try `pio device list` again
- **Upload fails**: Try pressing BOOT button on ESP32 during upload
- **Permission denied**: On Linux/macOS, you may need to add user to dialout group

### OTA Issues

- **Can't connect**: Verify device IP address is correct
- **Authentication fails**: Check `OTA_USERNAME` and `OTA_PASSWORD` match in both `secrets.h` and `platformio.ini`
- **Upload timeout**: Ensure device is on same network and WiFi is stable
- **Firmware too large**: Check flash size (this project uses 4MB partition table)

### WiFi Issues

- **Can't see setup AP**: Device may already be connected to WiFi - check router for device IP
- **Portal doesn't open**: Manually navigate to `http://192.168.4.1`
- **Credentials not saving**: Try resetting device (may need to clear flash)

---

## Quick Reference

### USB Flash
```bash
cd /Users/stas/Apps/home/ikea-led-obegraensad
pio run -e esp32dev -t upload
```

### OTA Flash (after configuring platformio.ini)
```bash
cd /Users/stas/Apps/home/ikea-led-obegraensad
pio run -e esp32dev -t upload
```

### Build Only (no upload)
```bash
pio run -e esp32dev
```

### Serial Monitor
```bash
pio device monitor
```

---

## Files to Edit

- **`include/secrets.h`**: OTA credentials (`OTA_USERNAME`, `OTA_PASSWORD`)
- **`include/constants.h`**: WiFi setup AP name (`WIFI_MANAGER_SSID`), pin configuration
- **`platformio.ini`**: OTA upload URL and credentials (for automated OTA)

---

## Notes

- **WiFi password is NOT stored in code** - configured via WiFi Manager portal
- **OTA credentials** (`OTA_USERNAME`/`OTA_PASSWORD`) are separate from WiFi credentials
- **Default OTA password**: `ikea-led-wall` (change in `secrets.h` for security)
- **Device hostname**: `ikea-led` (configurable in `secrets.h`)
