# 厂商资料索引

## 1. 官方产品页面

`https://www.lcdwiki.com/zh/3.5inch_ESP32-S3_Display`

页面正文的引脚分配表与厂商规格书一致，可作参考。页面内各文档的评估见第 2 节。

## 2. 官方文档索引

资源根：`https://www.lcdwiki.com/res/`

### 2.1 归档项

| 文档 | 链接 |
| --- | --- |
| 产品规格书 | `https://www.lcdwiki.com/res/ES3C35P/3.5inch_IPS_ESP32-S3%E4%BA%A7%E5%93%81%E8%A7%84%E6%A0%BC%E4%B9%A6_V1.0.pdf` |
| 用户手册 | `https://www.lcdwiki.com/res/ES3C35P/3.5inch_IPS_ESP32-S3%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C.pdf` |
| 原理图 | `https://www.lcdwiki.com/res/ES3C35P/ESP32-S3%E5%8E%9F%E7%90%86%E5%9B%BE.pdf` |
| 尺寸图 | `https://www.lcdwiki.com/res/ES3C35P/3.5inch_ESP32-S3_touch_Size.pdf` |
| 三维模型 | `https://www.lcdwiki.com/res/ES3C35P/3.5inch_ESP32-S3_board.zip` |
| 快速使用资料包 | `https://www.lcdwiki.com/res/ES3C35P/3.5inch_IPS_ESP32-S3_ES3C35P_Quick_Start.zip` |
| 快速使用手册 | `https://www.lcdwiki.com/res/ES3C35P/3.5inch_ESP32-S3_%E5%BF%AB%E9%80%9F%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C.pdf` |
| IO 资源分配表 | `https://www.lcdwiki.com/res/ES3C35P/ESP32-S3_IO%E8%B5%84%E6%BA%90%E5%88%86%E9%85%8D%E8%A1%A8.xlsx` |

快速使用资料包仅含预编译 bin 与 Windows 平台烧录工具，不含源码。

IO 资源分配表含 5 处错误，见第 4 节第 4 条与 [`01-hardware.md`](01-hardware.md) 第 6 节。

### 2.2 不适用项

| 文档 | 链接 | 原因 |
| --- | --- | --- |
| Arduino IDE 环境搭建 | `https://www.lcdwiki.com/res/PublicFile/ESP32_Arduino_IDE%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.pdf` | Windows 图形界面操作，与官方文档重复 |
| MicroPython 环境搭建 | `https://www.lcdwiki.com/res/PublicFile/ESP32_MicroPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.pdf` | 面向通用 ESP32-WROOM、4 MB Flash、CH340，与本模块不符 |
| VSCode + ESP-IDF 环境搭建 | `https://www.lcdwiki.com/res/PublicFile/%E4%BD%BF%E7%94%A8VSCode%E6%90%AD%E5%BB%BAESP-IDF%E7%8E%AF%E5%A2%83.pdf` | Windows 图形界面操作，与官方文档重复 |
| ESP-IDF LVGL 移植说明 | `https://www.lcdwiki.com/res/PublicFile/ESP-IDF_LVGL%E7%A7%BB%E6%A4%8D%E8%AF%B4%E6%98%8E.pdf` | 基于 `lvgl_esp32_drivers`（LVGL 8.3.11）与单线 SPI + DC 引脚方案，不适用于本模块的纯 QSPI 接口。其颜色配置说明（SPI 需交换 RGB565 字节、IPS 需反色）可作依据 |
| 小智 AI 快速使用手册 | `https://www.lcdwiki.com/res/ES3C28P/2.8inch_ESP32-S3%E5%B0%8F%E6%99%BAAI%E5%BF%AB%E9%80%9F%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C.pdf` | 该链接指向 2.8 英寸 E32C28P / E32N28P，非本模块 |

### 2.3 工具软件

路径前缀 `https://www.lcdwiki.com/res/software/`。该目录下均为通用工具，非本模块开发所必需。

| 文件 | 说明 |
| --- | --- |
| `Flash_Download.zip` | 烧录工具 |
| `JPGCompact_V5.0.zip` | JPEG 图片处理 |
| `TCP_UDP测试工具.zip` | 网络调试 |
| `串口调试助手.zip` | 串口调试 |
| `网络调试助手.zip` | 网络调试 |
| `PCtoLCD2002.zip` | 字符取模 |
| `Image2Lcd.zip` | 图片取模 |
| `esptouch-v2.0.0.apk` | Android 配网应用 |

Espressif 官方烧录工具不在此目录，另见 `https://dl.espressif.com/public/flash_download_tool.zip`。

## 3. 资料包

发放形式：

| 渠道 | 位置 |
| --- | --- |
| 百度网盘 | `https://pan.baidu.com/s/1niJLp_c5PqFeUwruDeT2Ug?pwd=an79` |
| 123pan | `https://1855123618.share.123pan.cn/123pan/Kg5Wvd-76Gr3?pwd=0SW3` |

目录结构：

```
1-示例程序_Demo/             Arduino 与 ESP-IDF 示例工程、依赖库
2-规格书_Specification/      产品规格书中英文版
3-尺寸图_Structure_Diagram/  尺寸图与 STEP 模型
4-数据手册_DataSheet/        ST77922、ES8311、TP4054、音频功放、WS2812B、
                            MEMS 麦克风、ESP32-S3 数据手册与硬件设计指南
                            （另含与本模块无关的 FT6336G 数据手册，见 3.2 节）
5-原理图_Schematic/          原理图与 IO 资源分配表
6-用户手册_User_Manual/      用户手册中英文版
7-工具软件_Tool_software/    通用工具
8-快速使用_Quick_Start/      预编译 bin 与烧录工具
```

### 3.1 核心文件

| 文件 | 内容 |
| --- | --- |
| `1-示例程序_Demo/Arduino/Install libraries/ST77922/ST77922.cpp` | 厂商 ST77922 初始化序列（63 条） |
| `1-示例程序_Demo/Arduino/Install libraries/ST77922/ST77922.h` | QSPI 引脚、操作码 `0x02` / `0x32`、时钟 80 MHz |
| `1-示例程序_Demo/ESP-IDF/3.5inch_ESP32-S3_LVGL/components/esp_bsp/bsp_display.c` | 厂商 ESP-IDF 显示 BSP，含同一初始化序列 |
| `1-示例程序_Demo/ESP-IDF/3.5inch_ESP32-S3_LVGL/components/esp_bsp/bsp_display.h` | 引脚、SPI 主机、时钟、背光 LEDC 参数 |
| `1-示例程序_Demo/ESP-IDF/3.5inch_ESP32-S3_LVGL/sdkconfig.defaults` | 参考 `sdkconfig`：Flash、PSRAM、CPU 频率、`CONFIG_LV_COLOR_16_SWAP` |
| `1-示例程序_Demo/ESP-IDF/3.5inch_ESP32-S3_LVGL/dependencies.lock` | 厂商验证的组件版本组合 |
| `4-数据手册_DataSheet/ST77922_SPEC_V1.2.pdf` | ST77922 数据手册 V1.2，245 页，含 QSPI 协议、命令表与接口时序（§6.4.5） |
| `4-数据手册_DataSheet/ST77922 TDDI Interface Protocol V01.00.pdf` | TDDI 接口协议 |
| `5-原理图_Schematic/ESP32-S3原理图.pdf` | 引脚、网络与器件型号依据 |

### 3.2 资料包内不适用文件

| 文件 | 原因 |
| --- | --- |
| `4-数据手册_DataSheet/D-FT6336G-DataSheet-V1.0.pdf` | FT6336G 为独立电容触摸控制器，使用自己的 I2C 从地址（非本模块的 0x55）。本模块的触摸控制器集成于 ST77922 内部，见 [`01-hardware.md`](01-hardware.md) 第 2.2 节。该文件与本模块无关，属资料包遗留；按 FT6336G 设计会得到错误的总线与从地址 |

### 3.3 示例工程

Arduino 侧：17 个依赖库（`Install libraries/` 下的目录数，与厂商 `3.5inch_arduino示例程序说明.pdf` 第 3.2 节逐条列出的库一致），29 个示例，覆盖显示（`Example_01`–`08`）、触摸（`15`、`28`）、音频（`16`、`17`）、WiFi（`18`–`24`）、BLE（`25`、`26`）、RGB 指示灯（`06`）、背光 PWM（`14`）、电池（`13`）、SD 卡（`05`）。

ESP-IDF 侧：`3.5inch_ESP32-S3_LVGL`，含自研 `components/esp_lv_port` 与 `components/esp_bsp`。

该示例使用 LVGL 8.4.0（API 为 `lv_disp_drv_t`、`LV_DISP_ROT_90`），非 LVGL 9。

## 4. 文档级勘误

| # | 文件 | 错误 |
| --- | --- | --- |
| 1 | `6-用户手册_User_Manual` 第 1 节 | 称 `2-规格书_Specification` 含"液晶屏显示驱动 IC 初始化代码"。该目录实际仅有 2 份规格书 PDF，初始化代码位于 `1-示例程序_Demo` |
| 2 | `1-示例程序_Demo/ESP-IDF/.../main/idf_component.yml` | 声明 `idf: ">=5.1"`，与同工程 `dependencies.lock` 锁定的 `esp_lcd_st77922` 1.0.3 及 IDF 5.4.2 不一致。组件 2.0.2 要求 IDF `>=5.4` |
| 3 | 官方页面「在本产品上部署小智AI的使用说明」链接 | 指向 2.8 英寸产品的文档 |
| 4 | `5-原理图_Schematic/ESP32-S3_IO资源分配表.xlsx` | 5 处错误，逐条列出见 [`01-hardware.md`](01-hardware.md) 第 6 节 |
| 5 | `6-用户手册_User_Manual` 第 3.2 节 (7) 与产品规格书第 4.1 节 | 对 IO38 / IO39 是否可作普通 IO 的表述互斥 |
| 6 | 音频功放器件 | 用户手册第 3.1 节 (15) 标注为 FM8002E，资料包数据手册为 `FM8002E.pdf`；原理图对应位置标注为 SC8002B。二者属 8002 系列单声道音频功放，封装与引脚兼容。实际 BOM 取值需向厂商确认 |
| 7 | 官方页面「ESP32主控参数」表 | 记「蓝牙：蓝牙V5.0 BR/EDR和蓝牙LE标准」。ESP32-S3 不支持 BR/EDR，仅支持 Bluetooth 5.0 LE |

## 5. 外部必备资料

| 资料 | 位置 |
| --- | --- |
| `espressif/esp_lcd_st77922` 组件 | `https://components.espressif.com/components/espressif/esp_lcd_st77922` |
| ST77922 数据手册 V0.1（较早版本） | `https://dl.espressif.com/AE/esp-iot-solution/ST77922_SPEC_V0.1.pdf` |
| ESP32-S3 数据手册 / 硬件设计指南 | 资料包 `4-数据手册`，或 Espressif 官方文档 |
| Espressif USB-Serial-JTAG 说明 | ESP-IDF 编程指南 |

同源硬件参考实现（QDtech ESP32-S3 3.5 英寸板，与本模块引脚及初始化序列一致）：

```
https://github.com/Liutupi/qdtech-s3-touch-lcd-3.5-xiaozhi-firmware
```
