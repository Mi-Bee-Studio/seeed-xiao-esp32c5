# Seeed XIAO ESP32-C5（感知节点主板）

[English](README.md) | [中文文档](README.zh.md)

[![Build Firmware](https://github.com/Mi-Bee-Studio/seeed-xiao-esp32c5/actions/workflows/build.yml/badge.svg)](https://github.com/Mi-Bee-Studio/seeed-xiao-esp32c5/actions/workflows/build.yml)

主板目录规范的一块板。**本仓库按"主板为根"规范组织**：

```
seeed-xiao-esp32c5/
├── README.md          # 本文件：这块板的一切硬件信息
└── <project>/         # 每个用这块板做的项目一个目录（按能力命名）
    ├── CMakeLists.txt / main/ / sdkconfig.defaults / main/idf_component.yml
    └── README.md      # 项目说明 + 编译/烧录命令
```

规范要点：

- **主板目录名** = 板子名（kebab-case），根 README 只写硬件、不写业务；
- **项目目录独立可编译**：自带完整构建三件套，`cd <project> && idf.py build` 即出固件；
- 项目间不共享代码；需要共性时先拷贝，稳定后再考虑抽组件。

### 固件基线规范（全家桶强制）

1. **看门狗：必须启用**。任务订阅 ESP-IDF TWDT 按周期喂狗；
2. **Web/API 固件升级（OTA）：硬件允许则必须提供**。本板有双频 WiFi、8MB flash 放得下 OTA 双槽，每个项目都要带板端 web 刷机能力。

| 项目 | 看门狗 | Web/API OTA |
|------|--------|-------------|
| blink | ✅ 任务订阅 TWDT（5s panic） | ✅ OTA 双槽 + `POST /ota` 流式写槽 |

---

## 板子概要

| 项目 | 值 |
|------|-----|
| 芯片 | **ESP32-C5** —— RISC-V 双核 240MHz，384KB SRAM / 320KB ROM（裸芯片，非模组） |
| Flash | **8MB 外挂**（Puya PY25Q64HA，QSPI） |
| PSRAM | wiki 口径 **8MB Flash & 8MB PSRAM**（in-package PSRAM 变体；本仓未实测，启用 `SPIRAM` 前先真机验证） |
| 无线 | **双频 WiFi 6（2.4GHz + 5GHz，802.11 a/b/g/n/ac/ax）** + Bluetooth 5 (LE) |
| USB | 原生 USB Type-C（**USB-Serial-JTAG**：控制台/烧录同一口，无桥芯片） |
| 板载 LED | **User LED = GPIO27**（单色，丝印 `L`）+ 红色充电指示（丝印 `C`） |
| 按键 | BOOT = **GPIO28**（按住上电进下载）、RESET = CHIP_EN |
| 天线 | 外接 **U.FL** 天线座（双频天线开关 Skyworks LFD182G45DCHD277） |
| 尺寸 | 21 × 17.8 mm（标准 XIAO 形制） |
| IDF | **v5.5 起支持（preview 口径）**；本仓 CI/构建用 **v6.0**（v6.0 勘误：启用 flash encryption 时 CPU 暂限 160MHz） |

## 引脚位置图（USB-C 朝上，正面/元件面视角；D 编号 = XIAO 丝印）

```
                 ┌─ USB-C ─┐
       5V ◎┬───┘          ├───┬◎ D0 = GPIO1  (ADC, LP_UART_DSRN)
      GND ◎│              │   ◎ D1 = GPIO0  (LP_UART_DTRN)
      3V3 ◎│  [USER LED]  │   ◎ D2 = GPIO25
     D8 ◎  │   = GPIO27   │   ◎ D3 = GPIO7  (SDIO_DATA1)
     D9 ◎  │              │   ◎ D4 = GPIO23 (I2C SDA)
    D10 ◎  │   ESP32-C5   │   ◎ D5 = GPIO24 (I2C SCL)
           │              │   ◎ D6 = GPIO11 (UART TX)
           └──────────────┘   ◎ D7 = GPIO12 (UART RX)
            左列（正面）    右列（正面）

  背面 JTAG 焊盘：MTDO=GPIO5 · MTDI=GPIO3 · MTCK=GPIO4 · MTMS=GPIO2
  电池检测：ADC_BAT=GPIO6（读数×2.0）· 使能=GPIO26
```

要点（引脚映射出处：[Seeed wiki Pin Map](https://wiki.seeedstudio.com/xiao_esp32c5_getting_started/)）：

- **左列**自 USB 端向下：`5V · GND · 3V3 · D8=GPIO8(SPI SCK) · D9=GPIO9(SPI MISO) · D10=GPIO10(SPI MOSI)`；
- **右列**自 USB 端向下：`D0=GPIO1 · D1=GPIO0 · D2=GPIO25 · D3=GPIO7 · D4=GPIO23(SDA) · D5=GPIO24(SCL) · D6=GPIO11(TX) · D7=GPIO12(RX)`；
- 物理排布以 Seeed 官方 pinout 图为准；本图按"USB-C 朝上"标准视角整理。

## 注意事项

- **无 USB-UART 桥**：串口即 C5 的 USB-Serial-JTAG；
- BOOT = GPIO28（注意与 C6 的 GPIO9 不同），按住插 USB / 按 RESET 进下载模式；
- **双频是这块板的差异化卖点**（2.4G 拥挤/远距 5G 场景），后续项目可用；
- PSRAM 与 flash encryption 相关勘误未实测——启用前先真机验证；
- IDF 对 C5 的支持自 v5.5 为 preview 口径，跨版本行为差异注意对照 release notes。

## 项目索引

| 项目 | 说明 |
|------|------|
| [blink](blink/README.zh.md) | 基线工程/测试固件：USER LED（GPIO27）心跳闪烁 + BOOT 交互 + web 维护页（配网/OTA）+ TWDT 看门狗 |
