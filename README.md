# Zigbee Sniffer & Wireshark Demo

This repository documents how to sniff, decrypt, and analyze Zigbee traffic with:

- **Coordinator:** Sonoff Zigbee 3.0 USB Dongle Plus (CC2652P)
- **Sniffer:** Nordic nRF52840 Dongle running nRF Sniffer firmware
- **Devices:** Sonoff Zigbee Button & Zigbee Light Sensor
- **Software:** Zigbee2MQTT, Mosquitto, Wireshark

The goal is to show how Zigbee communication (e.g., button press, sensor update) looks in Wireshark once properly decrypted.

---

## 📖 Contents

- `docs/` &rarr; Step-by-step guides:
    - Flashing & setting up the sniffer
    - Wireshark configuration with Zigbee keys
    - Running Zigbee2MQTT with Docker
    - Device behavior & interpretation
- `images/` &rarr; Topology diagrams, screenshots, device photos
- `scripts/` &rarr; Docker Compose & helper scripts

---

## 🚀 Quick Start

### 1. Coordinator setup

Use the **Sonoff Zigbee 3.0 USB Dongle Plus** as your Zigbee coordinator with Zigbee2MQTT.

You maybe need to flash the latest Zigbee2MQTT firmware onto the dongle. I used this link:
[Sonoff Firmware](https://dongle.sonoff.tech/sonoff-dongle-flasher/)

Then, with the Docker Compose file in `scripts/`, start Zigbee2MQTT and Mosquitto:

```bash
# I did this in WSL (Ubuntu) on Windows
docker compose up -d
```

Then, you can access the Zigbee2MQTT frontend at `http://localhost:8080` and pair your devices (Sonoff Button, Light Sensor).

### 2. Sniffer setup

Flash the Nordic nRF52840 Dongle with Nordic’s nRF Sniffer for 802.15.4.  
Connect it to Wireshark via extcap.

I used this repo for flashing and setup:
[nRF Sniffer for 802.15.4](https://github.com/NordicSemiconductor/nRF-Sniffer-for-802.15.4)

### 3. Wireshark configuration


In **Edit → Preferences → Protocols → IEEE 802.15.4 → Decryption Keys**:

- Add a new key with:
  - Decryption key: `00112233445566778899AABBCCDDEEFF` (example, use your own)
  - Decryption key index: `0`
  - Key hash: `No hash`

> [!NOTE]
> You'll find the Zigbee Network Key in `coordinator_backup.json` in the `data` folder created by Zigbee2MQTT.

In **Edit → Preferences → Protocols → ZigBee**:

- Add the Network Key:
    - Key: `00:11:22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF` (example, use your own, colon-separated)
    - Byte Order: `Normal`
    - Label: `Zigbee Network Key` (for reference)

Now Wireshark will show decrypted ZCL frames when you press the button.

---

### 🖼️ Example

**Button single press:**

**MQTT:**
```json
{"action":"single","battery":100,"linkquality":252}
```

**Wireshark:** shows decrypted ZCL frame with manufacturer-specific payload.

See `images/double_press.png`.

---

### 🛠️ Tools

- Zigbee2MQTT
- Mosquitto
- Wireshark
- nRF Sniffer for 802.15.4

---

### ⚠️ Disclaimer

This setup is for educational purposes. Don’t sniff or analyze Zigbee traffic from networks you don’t own.

---

## 🖥️ Platform Notes

All steps in this project were done on **Windows 10/11**, except for **Zigbee2MQTT**, which I ran inside **WSL (Ubuntu)** because it is not natively supported on Windows.

To pass the Sonoff Zigbee 3.0 USB Dongle Plus into WSL, I used [`usbipd-win`](https://github.com/dorssel/usbipd-win):

### PowerShell

```powershell
# With this command you can see the connected USB devices and their BUSID
usbipd list
# Here I put 2-6 as an example, use the correct busid for your dongle
usbipd bind --busid 2-6
usbipd attach --wsl --busid 2-6
```

### Ubuntu (WSL)

```bash
# and in WSL you can check if the device is available
ls /dev/ttyUSB*
```

This makes the dongle accessible inside WSL as /dev/ttyUSB0, which Zigbee2MQTT can use as its serial adapter.