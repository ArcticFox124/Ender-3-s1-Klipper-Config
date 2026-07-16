# Ender 3 S1 Klipper Config (STM32F401)

A complete guide to flashing Klipper on the Ender 3 S1 with a **STM32F401 mainboard**. Most guides online are written for the F103 board and skip critical steps for the F401. This guide is tested and working.

---

## Hardware

- Creality Ender 3 S1
- Mainboard: STM32F401 (CR4NT V2.2)
- Raspberry Pi 3B+
- CR Touch

---

## Prerequisites

- Raspberry Pi flashed with Raspberry Pi OS Lite
- SSH access to the Pi
- SD card (FAT32, 32GB or less)

---

## Step 1 — Install Klipper, Moonraker, and Mainsail

Use KIAUH (Klipper Installation And Update Helper):

```bash
sudo apt-get update && sudo apt-get install git -y
cd ~
git clone https://github.com/dw-0/kiauh.git
./kiauh/kiauh.sh
```

Inside KIAUH install in this order:
1. Klipper
2. Moonraker
3. Mainsail

---

## Step 2 — Compile Klipper Firmware

```bash
cd ~/klipper
make menuconfig
```

Set these exact options:

```
Enable extra low-level configuration options  →  [*]
Micro-controller Architecture                 →  STMicroelectronics STM32
Processor model                               →  STM32F401
Bootloader offset                             →  64KiB bootloader
Clock Reference                               →  8 MHz crystal
Communication interface                       →  Serial (on USART1 PA10/PA9)
Baud rate                                     →  250000
Optimize stepper code for step on both edges  →  [*]
```

Press `Q` then `Y` to save. Then build:

```bash
make
```

Firmware will be at `~/klipper/out/klipper.bin`.

---

## Step 3 — Flash the Firmware

This is the step most guides get wrong for the F401.

1. Copy `~/klipper/out/klipper.bin` to your SD card
2. Rename it to `firmware.bin`
3. Make sure the SD card is formatted **FAT32**
4. Insert the SD card into the **front SD slot** (the same one you use for gcode files)
5. Power on the printer and wait **60 seconds**
6. Power off the printer
7. Remove the SD card
8. Power back on

> ⚠️ **Important:** The front SD slot is the correct slot for flashing on the F401 board. Not the microSD slot on the display board. Not a separate mainboard slot. The normal front gcode SD slot.

> ⚠️ **Note:** After flashing Klipper, the touchscreen will be stuck on the Creality logo forever. This is normal — you control the printer through Mainsail in your browser now.

---

## Step 4 — Connect Printer to Pi

Connect the printer to the Pi via USB cable. Then find the serial port:

```bash
ls /dev/serial/by-id/
```

Copy the device path — you'll need it for `printer.cfg`.

---

## Step 5 — Configure printer.cfg

Copy the `printer.cfg` from this repo into your Mainsail config directory:

```bash
cp printer.cfg ~/printer_data/config/printer.cfg
```

Update the `[mcu]` serial line with your device path:

```ini
[mcu]
serial: /dev/serial/by-id/YOUR_DEVICE_ID_HERE
```

Then in Mainsail click **Save & Restart**.

---

## Step 6 — First Time Calibration

Once Klipper is connected run these in order via the Mainsail console:

1. `G28` — home all axes
2. `SCREWS_TILT_CALCULATE` — level the bed
3. `PROBE_CALIBRATE` — set Z offset
4. `BED_MESH_CALIBRATE` — map the bed mesh
5. `SAVE_CONFIG` — save everything

---

## Troubleshooting

**Stuck on Creality logo after flash** — This is normal with Klipper. The touchscreen doesn't work with Klipper. Use Mainsail.

**SD card not being read for flashing** — Make sure it's FAT32, not exFAT. 32GB cards on Windows often default to exFAT — manually select FAT32 when formatting.

**firmware.bin still on SD card after flash attempt** — The bootloader didn't read it. Check FAT32 format and make sure you're using the front SD slot.

**CH340 showing in /dev/serial/by-id/** — The firmware may still be working via serial. Check Mainsail — if it's connected and printing, you're fine.

---

## Credits

Guide written from real-world troubleshooting experience flashing the STM32F401 variant of the Ender 3 S1. Most existing guides don't cover this board revision accurately.
