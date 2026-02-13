

# 🖨️ GuguBOT 3D打印机 Klipper 配置文件  
*高性能CoreXY结构打印机，采用CAN总线架构，专为速度与可靠性深度优化*

![GuguBOT打印效果](https://github.com/changchuanyong/3dPrinter/raw/main/resources/gugubot.jpg)  
*（本配置实测打印效果）*

## 🔧 硬件配置清单
| 组件             | 详细说明                                                                 |
|------------------|--------------------------------------------------------------------------|
| **主控板**       | MKS Monster8 V2.0 (STM32F407) + CAN总线                                  |
| **喷头控制板**   | BIGTREETECH EBB Can V1.2 (STM32G0B1)                                     |
| **结构类型**     | CoreXY + 三Z轴独立调平（3组Z电机）                                       |
| **热床**         | 300×300mm 加热平台（最高95°C）                                           |
| **挤出系统**     | BMG齿轮挤出机 + TMC2209静音驱动                                          |
| **传感器**       | ADXL345加速度计、高精度探针（Z偏移：0.55mm）                             |
| **摄像头**       | 640×480 USB摄像头 @ 15FPS（crowsnest流媒体支持）                         |
| **网络**         | WiFi + sonar守护进程（自动断线重连）                                     |

## 📦 软件生态
- **固件核心**：[Klipper](https://github.com/Klipper3d/klipper)（含输入整形优化）
- **操作界面**：[MainsailOS](https://github.com/mainsail-crew/mainsail-os)
- **视频流**：[crowsnest](https://github.com/mainsail-crew/crowsnest)（多路摄像头管理）
- **网络守护**：[sonar](https://github.com/mainsail-crew/sonar)（WiFi稳定性保障）
- **延时摄影**：[moonraker-timelapse](https://github.com/mainsail-crew/moonraker-timelapse)

## ⚡ 性能调优亮点
```ini
# CoreXY运动参数
max_velocity: 500 mm/s
max_accel: 20,000 mm/s²
square_corner_velocity: 5.0 mm/s

# 输入整形（实测校准值）
shaper_type_x: mzv @ 87.4Hz
shaper_type_y: 2hump_ei @ 66.8Hz

# 压力提前量
pressure_advance: 0.06（PLA材料优化）
```

## 🎥 摄像头流媒体配置 (crowsnest)
- **实时预览**：`http://<打印机IP>:8080/webcam/?action=stream`
- **截图接口**：`http://<打印机IP>:8080/webcam/?action=snapshot`
- **RTSP流**：默认关闭（需在`crowsnest.conf`中启用）
- **分辨率**：640×480 @ 15FPS（支持按摄像头能力调整）

![Mainsail摄像头界面](https://github.com/changchuanyong/3dPrinter/raw/main/resources/webcam-view.jpg)  
*Mainsail界面中的实时监控画面*

## 🔑 核心功能特性
✅ **智能调平系统**  
- 三Z轴自动对齐（`Z_TILT_ADJUST`）  
- 6×6双三次样条网床补偿  
- 温度自适应配置文件（PLA 60°C / PETG 80°C）  

✅ **可靠性增强**  
- sonar守护进程自动恢复WiFi连接  
- 断料检测（EBB Can传感器）  
- 全加热器热失控保护  

✅ **高效工作流宏**  
```gcode
PRINT_START BED=60 EXTRUDER=205  ; 一键启动：归零→调平→网床校准
M600                            ; 智能换料（自动回抽+移位）
Z_TILT_ADJUST                   ; 三Z轴同步校准
```

✅ **远程管理**  
- Mainsail全功能Web界面（端口80）  
- Moonraker API兼容OctoPrint生态  
- 一键固件/软件更新（Update Manager）

## ⚠️ 重要配置说明
1. **CAN总线设备识别**  
   请替换配置中的UUID为实际设备ID：
   ```ini
   [mcu]
   canbus_uuid: 313c75eb5119  # ← 执行 `ls /dev/serial/by-id/*` 查询
   
   [mcu EBBCan]
   canbus_uuid: 384c321d1a11  # ← 执行 `sudo ip -br link show can0` 查询
   ```

2. **探针物理偏移校准**  
   必须根据实际测量值调整：
   ```ini
   [probe]
   x_offset: 20.7  # 喷嘴中心到探针尖端的X向距离（单位：mm）
   y_offset: 6.5   # Y向距离（影响网床精度的关键参数）
   z_offset: 0.550 # 通过 PROBE_CALIBRATE 精确设定
   ```

3. **温控参数重校准**  
   不同硬件需重新PID整定：
   ```gcode
   PID_CALIBRATE HEATER=extruder TARGET=250
   PID_CALIBRATE HEATER=bed TARGET=70
   SAVE_CONFIG
   ```

## 🛠️ 部署指南
1. **环境准备**  
   - 树莓派3B+/4 运行 [MainsailOS](https://github.com/mainsail-crew/MainsailOS)  
   - MKS Monster8 V2.0 已刷入 [Klipper CAN Bootloader](https://www.klipper3d.org/Can_Bootloader.html)  
   - EBB Can V1.2 已刷入 [Klipper固件](https://github.com/bigtreetech/EBB)  

2. **配置部署**  
   ```bash
   # 在树莓派终端执行
   cd ~/printer_data/config
   git clone https://github.com/changchuanyong/3dPrinter.git
   ln -s ./3dPrinter/配置文件/* ./
   sudo service klipper restart
   ```

3. **首次启动流程**  
   ```gcode
   CALIBRATE_ACCEL  ; 测量共振频率（生成输入整形参数）
   Z_TILT_ADJUST    ; 三Z轴机械同步
   BED_MESH_CALIBRATE ; 创建热变形补偿网格
   SAVE_CONFIG      ; 永久保存配置
   ```

## 🔧 维护与更新
所有组件支持Moonraker一键更新：  
进入Mainsail → **机器 → 更新管理器** 即可完成：
- Klipper固件
- Mainsail界面
- crowsnest摄像头服务
- sonar网络守护
- 延时摄影插件

## 📜 免责声明
> 本配置专为 **GuguBOT打印机 + MKS Monster8 V2.0主控 + EBB Can V1.2喷头板** 组合优化。  
> **使用前务必注意**：  
> 1️⃣ 核实温度传感器型号与硬件完全匹配  
> 2️⃣ 首次上电前确认步进电机接线正确  
> 3️⃣ 初次测试建议降低驱动电流（run_current）  
> 4️⃣ 首次打印全程人工监护  
> *配置不当可能导致设备损坏或安全事故*

---

💡 **专业提示**：  
- 搭配 [KlipperScreen](https://github.com/KlipperScreen/KlipperScreen) 实现本地触控操作  
- 长时间打印遇WiFi不稳定？启用 `sonar.conf` 中的守护进程  
- 机械结构变动后（如更换皮带/导轨），务必重新校准输入整形参数  

[![Mainsail操作界面](https://github.com/changchuanyong/3dPrinter/raw/main/resources/mainsail-ui.jpg)](https://github.com/mainsail-crew/mainsail)  
*集成摄像头监控的Mainsail操作界面*

> **仓库结构说明**  
> ```
> ├── printer.cfg             # Klipper主配置文件
> ├── crowsnest.conf          # 摄像头流媒体配置
> ├── sonar.conf              # WiFi守护进程配置
> ├── macros.cfg              # 自定义G代码宏集合
> ├── ebb-canbus-v1.2.cfg     # EBB Can喷头板定义
> ├── mainsail.cfg            # Web界面预设参数
> └── resources/              # 校准示意图/打印效果参考
> ```

⭐ **觉得有用？欢迎Star支持！**  
欢迎提交PR适配您的硬件，共同完善开源配置生态  
*本配置已在GuguBOT v2.1实测：0.4mm喷嘴，210°C PLA打印* 🌱
