



# 🖨️ GuguBOT 3D Printer Klipper Configuration  
*High-performance CoreXY printer with CAN bus architecture, optimized for speed and reliability*

![GuguBOT Printer](https://github.com/changchuanyong/3dPrinter/raw/main/resources/gugubot.jpg)  
*(Example print from this configuration)*

## 🔧 Hardware Specifications
| Component          | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Main Controller** | MKS Monster8 V2.0 (STM32F407) with CAN bus                              |
| **Toolhead Board**  | BIGTREETECH EBB Can V1.2 (STM32G0B1)                                    |
| **Kinematics**      | CoreXY with triple-Z (3 independent Z motors)                          |
| **Bed**             | 300×300mm heated bed (max 95°C)                                         |
| **Extruder**        | BMG-style geared extruder with TMC2209 stealthChop                      |
| **Sensors**         | ADXL345 accelerometer, Precision probe (Z-offset: 0.55mm)              |
| **Camera**          | 640×480 USB webcam @ 15FPS (streaming via crowsnest)                    |
| **Connectivity**    | WiFi with automatic recovery (sonar daemon)                            |

## 📦 Software Stack
- **Firmware**: [Klipper](https://github.com/Klipper3d/klipper) (with input shaping)
- **Companion**: [MainsailOS](https://github.com/mainsail-crew/mainsail-os)
- **Camera System**: [crowsnest](https://github.com/mainsail-crew/crowsnest)
- **Network Guard**: [sonar](https://github.com/mainsail-crew/sonar) (WiFi keepalive)
- **Timelapse**: [moonraker-timelapse](https://github.com/mainsail-crew/moonraker-timelapse)

## ⚡ Performance Optimizations
```ini
# CoreXY Motion Parameters
max_velocity: 500 mm/s
max_accel: 20,000 mm/s²
square_corner_velocity: 5.0 mm/s

# Input Shaping (Auto-calibrated)
shaper_type_x: mzv @ 87.4Hz
shaper_type_y: 2hump_ei @ 66.8Hz

# Pressure Advance
pressure_advance: 0.06 (PLA optimized)

# Vibration Damping
[tmc2209] drivers in stealthChop mode for all axes
```

## 🎥 Camera Streaming (crowsnest)
- **Web Interface**: `http://<printer-ip>:8080/webcam/?action=stream`
- **Snapshot URL**: `http://<printer-ip>:8080/webcam/?action=snapshot`
- **RTSP Stream**: Disabled by default (enable in `crowsnest.conf`)
- **Resolution**: 640×480 @ 15FPS (adjustable for higher-end cameras)

![Camera View](https://github.com/changchuanyong/3dPrinter/raw/main/resources/webcam-view.jpg)  
*Live webcam view in Mainsail UI*

## 🔧 Key Features
1. **Intelligent Bed Leveling**  
   - 3-Z motor auto-alignment (`Z_TILT_ADJUST`)
   - 6×6 bicubic mesh bed compensation
   - Thermal-aware profiles (PLA 60°C / PETG 80°C)

2. **Reliability Systems**  
   - WiFi auto-recovery daemon (sonar)
   - Filament runout detection (EBB Can sensor)
   - Thermal runaway protection on all heaters

3. **Quality-of-Life Macros**  
   ```gcode
   PRINT_START BED=60 EXTRUDER=205  ; Auto bed leveling + mesh calibration
   M600                            ; Filament change with parking routine
   Z_TILT_ADJUST                   ; Triple-Z motor synchronization
   ```

4. **Remote Management**  
   - Full Mainsail web interface (port 80)
   - Moonraker API for OctoPrint compatibility
   - One-click firmware updates via update manager

## ⚠️ Critical Configuration Notes
1. **CAN Bus Setup**  
   Replace UUIDs in configuration with your actual device IDs:
   ```ini
   [mcu]
   canbus_uuid: 313c75eb5119  # ← Run `ls /dev/serial/by-id/*` to find yours
   
   [mcu EBBCan]
   canbus_uuid: 384c321d1a11  # ← Check `sudo ip -br link show can0`
   ```

2. **Probe Calibration**  
   Physical offsets must be calibrated:
   ```ini
   [probe]
   x_offset: 20.7  # Measured from nozzle center to probe tip
   y_offset: 6.5   # Critical for accurate mesh generation
   z_offset: 0.550 # Set via PROBE_CALIBRATE
   ```

3. **Thermal Settings**  
   PID values are printer-specific - recalibrate after assembly:
   ```gcode
   PID_CALIBRATE HEATER=extruder TARGET=250
   PID_CALIBRATE HEATER=bed TARGET=70
   ```

## 🛠️ Installation Guide
1. **Prerequisites**  
   - Raspberry Pi 3B+/4 running [MainsailOS](https://github.com/mainsail-crew/MainsailOS)
   - MKS Monster8 V2.0 flashed with [Klipper CAN bootloader](https://www.klipper3d.org/Can Bootloader.html)
   - EBB Can V1.2 flashed with [Klipper firmware](https://github.com/bigtreetech/EBB)

2. **Deployment**  
   ```bash
   # On Raspberry Pi:
   cd ~/printer_data/config
   git clone https://github.com/changchuanyong/3dPrinter.git
   ln -s ./3dPrinter/配置文件/* ./
   sudo service klipper restart
   ```

3. **First Boot Sequence**  
   ```gcode
   CALIBRATE_ACCEL  ; Measures resonant frequencies
   Z_TILT_ADJUST    ; Aligns Z motors
   BED_MESH_CALIBRATE ; Creates thermal compensation map
   SAVE_CONFIG      ; Writes settings to config
   ```

## 🔧 Maintenance & Updates
All components support in-place updates via Moonraker:
```ini
[update_manager]
# Managed components:
# - Klipper core firmware
# - Mainsail web interface
# - Crowsnest camera system
# - Sonar WiFi daemon
# - Timelapse generator
```
Access updates through the **Machine → Update Manager** tab in Mainsail.

## 📜 Disclaimer
> This configuration is specifically tuned for **GuguBOT printers** with MKS Monster8 V2.0 mainboards and EBB Can V1.2 toolhead controllers. Using it on different hardware may cause damage. Always:
> 1. Verify thermal sensor types match your hardware
> 2. Confirm stepper motor wiring before powering on
> 3. Start with reduced motor currents during initial tests
> 4. Supervise first prints closely

---

**💡 Pro Tip:** For best results with this configuration:  
- Use [KlipperScreen](https://github.com/KlipperScreen/KlipperScreen) on a Raspberry Pi display for local control  
- Enable `sonar` in `sonar.conf` if experiencing WiFi dropouts during long prints  
- Calibrate input shaper monthly or after major mechanical changes  

[![Mainsail Dashboard](https://github.com/changchuanyong/3dPrinter/raw/main/resources/mainsail-ui.jpg)](https://github.com/mainsail-crew/mainsail)  
*Control interface with real-time webcam integration*

> **Repository Structure**  
> ```
> ├── printer.cfg             # Main Klipper configuration
> ├── crowsnest.conf          # Webcam streaming settings
> ├── sonar.conf              # WiFi reliability daemon
> ├── macros.cfg              # Custom G-code macros
> ├── ebb-canbus-v1.2.cfg     # EBB Can toolhead definitions
> ├── mainsail.cfg            # Web interface presets
> └── resources/              # Calibration photos and diagrams
> ```

⭐ **Star this repo if you find it useful!** Contributions and hardware-specific adaptations welcome.  
*Config tested on GuguBOT v2.1 with 0.4mm nozzle @ 210°C PLA printing*
