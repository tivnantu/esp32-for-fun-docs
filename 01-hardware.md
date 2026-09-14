# 硬件

## 1. 模组

| 项 | 值 | 来源 |
| --- | --- | --- |
| 主控 | ESP32-S3R8 | [实测+交叉] |
| CPU | Xtensa LX7 双核，最高 240 MHz | [文档] |
| ROM / SRAM / RTC SRAM | 384 KB / 512 KB / 16 KB | [文档] |
| PSRAM | 8 MB，封装内，OPI（八线） | [实测+交叉] |
| Flash | 16 MB，外接，QSPI | [实测+交叉] |
| 无线 | 2.4 GHz 802.11b/g/n；蓝牙 5.0 LE。不含 BR/EDR（厂商文档误记，见 [`08-reference-index.md`](08-reference-index.md) 第 4 节第 7 条） | [勘误] |
| 芯片工作电压 | 3.0 ~ 3.6 V | [文档] |

模组为 N16R8（封装内 8 MB OPI PSRAM，外接 16 MB QSPI Flash）。参考配置：

```
CONFIG_ESPTOOLPY_FLASHSIZE_16MB=y
CONFIG_ESPTOOLPY_FLASHMODE_QIO=y
CONFIG_SPIRAM=y
CONFIG_SPIRAM_MODE_OCT=y
CONFIG_SPIRAM_SPEED_80M=y
CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ_240=y
```

Flash 模式 DIO 亦可启动（兼容子集，占用引脚更少）。

### 1.1 器件型号

| 器件 | 型号 | 来源 |
| --- | --- | --- |
| 屏组件 | HMX035CTFT-001 | [文档] |
| 音频编解码 | ES8311 | [文档] |
| 音频功放 | 8002 系列单声道，型号标注存在冲突，见 [`08-reference-index.md`](08-reference-index.md) 第 4 节第 6 条 | [文档] |
| MEMS 麦克风 | LMA2718B381-OA7 | [文档] |
| RGB 指示灯 | XL-5050RGBC-WS2812B | [文档] |
| 电池充电管理 | TP4054 | [文档] |
| 稳压 | ME6217C33M5G | [文档] |
| 外接 QSPI Flash | 16 MB，厂商原理图未标注型号 | — |

型号取自厂商原理图。屏组件型号标注于显示触摸屏 FPC 连接器处。

## 2. 引脚分配

### 2.1 面板（QSPI）

| 信号 | GPIO | 说明 | 来源 |
| --- | --- | --- | --- |
| CS | IO10 | 片选，低有效 | [实测+交叉] |
| PCLK | IO12 | QSPI 时钟 | [实测+交叉] |
| D0 | IO11 | 数据线 | [实测+交叉] |
| D1 | IO13 | 数据线 | [实测+交叉] |
| D2 | IO14 | 数据线 | [实测+交叉] |
| D3 | IO9 | 数据线 | [实测+交叉] |
| RST | EN（CHIP_PU） | 与主控复位共用，无独立 GPIO | [实测+交叉] |
| BL | IO41 | 背光，高电平点亮，默认低（灭） | [实测+交叉] |

该接口无 DC（命令/数据选择）信号。命令与数据的区分由 QSPI 协议的操作码完成，见 [`03-display-st77922.md`](03-display-st77922.md)。

### 2.2 电容触摸

| 信号 | GPIO | 说明 | 来源 |
| --- | --- | --- | --- |
| I2C SDA | IO38 | 触摸 I2C 数据；总线复用见 2.8 | [实测+交叉] |
| I2C SCL | IO39 | 触摸 I2C 时钟；总线复用见 2.8 | [实测+交叉] |
| RST | IO48 | 低有效 | [实测+交叉] |
| INT | IO47 | 低有效 | [实测+交叉] |
| I2C 从地址 | — | 0x55 | [文档] |

触摸控制器集成于 ST77922 内部。

### 2.3 MicroSD（SDIO 4 位）

| 信号 | GPIO | 来源 |
| --- | --- | --- |
| CLK | IO5 | [实测+交叉] |
| CMD | IO4 | [实测+交叉] |
| DATA0 | IO6 | [实测+交叉] |
| DATA1 | IO7 | [实测+交叉] |
| DATA2 | IO2 | [实测+交叉] |
| DATA3 | IO3 | [实测+交叉] |

### 2.4 音频

| 信号 | GPIO | 说明 | 来源 |
| --- | --- | --- | --- |
| 功放使能 | IO1 | 低电平使能，高电平禁止 | [实测+交叉] |
| I2S MCLK | IO17 | 主时钟 | [实测+交叉] |
| I2S BCLK | IO18 | 位时钟 | [实测+交叉] |
| I2S DOUT | IO15 | 数据输出（ESP32-S3 → 编解码） | [实测+交叉] |
| I2S DIN | IO16 | 数据输入（编解码 → ESP32-S3） | [实测+交叉] |
| I2S WS/LRCK | IO21 | 左右声道选择，高=右，低=左 | [实测+交叉] |
| 编解码 I2C | IO38 / IO39 | 与触摸共用 | [实测+交叉] |
| 编解码从地址 | — | 0x18 | [文档] |

编解码芯片为 ES8311；功放为 8002 系列单声道音频功放，最大 1.5 W（8 Ω）或 2 W（4 Ω）。器件型号标注存在冲突，见 [`08-reference-index.md`](08-reference-index.md) 第 4 节第 6 条。

### 2.5 按键 · 串口 · USB

| 信号 | GPIO | 说明 | 来源 |
| --- | --- | --- | --- |
| BOOT 按键 | IO0 | 上电时拉低进入下载模式；其余情况可作普通按键 | [实测+交叉] |
| RESET 按键 | EN | 低电平复位，与面板复位共用 | [实测+交叉] |
| UART0 TXD | IO43 | 芯片内定义 `U0TXD` | [实测+交叉] |
| UART0 RXD | IO44 | 芯片内定义 `U0RXD` | [实测+交叉] |
| USB D- | IO19 | 与 Type-C 座相连，复用原生 USB-Serial-JTAG | [实测+交叉] |
| USB D+ | IO20 | 同上 | [实测+交叉] |

芯片侧方向：`U0TXD`（GPIO43）为输出，`U0RXD`（GPIO44）为输入。厂商引脚表以板级网络名标注为 `RXD0(IO43)` / `TXD0(IO44)`，与芯片内部信号方向相反；接线以芯片定义为准。

### 2.6 扩展

| 信号 | GPIO | 说明 | 来源 |
| --- | --- | --- | --- |
| 扩展 IO | IO45 | 引出至 1.25 mm 4P 座。同时为启动配置引脚，见 2.9 | [实测+交叉] |
| 扩展 IO | IO46 | 同上。同时为启动配置引脚，见 2.9 | [实测+交叉] |
| 外部 I2C SDA | IO38 | 与触摸、编解码共用 | [实测+交叉] |
| 外部 I2C SCL | IO39 | 同上 | [实测+交叉] |
| 电池检测 | IO8 | ADC 输入 | [实测+交叉] |
| RGB 指示灯 | IO40 | 单线，内置控制 IC（XL-5050RGBC-WS2812B） | [实测+交叉] |

### 2.7 不可用 IO

| GPIO | 占用 |
| --- | --- |
| IO26 | PSRAM 片选 |
| IO27 – IO32 | Flash 与 PSRAM 共用（SPIHD / SPIWP / SPICS0 / SPICLK / SPIQ / SPID） |
| IO33 – IO37 | PSRAM 数据线 |

上述引脚未引出，不可作普通 IO。

### 2.8 复用约束

| GPIO | 约束 |
| --- | --- |
| IO38 / IO39 | 触摸 I2C、ES8311 I2C、外部 I2C 扩展口共用同一总线。使用其中任一功能时不可作普通 IO；三者均不使用时可作普通 IO |
| IO40 | 仅用于 RGB 指示灯 |
| IO41 | 仅用于背光控制 |
| IO47 / IO48 | 仅用于触摸中断与复位 |
| IO42 | 官方 BSP 未使用；厂商引脚表误标为液晶屏命令/数据选择控制，见第 6 节第 3 条 |

### 2.9 启动配置（strapping）引脚

ESP32-S3 共 4 个 strapping 引脚：GPIO0、GPIO3、GPIO45、GPIO46。四者在 CHIP_PU 上升沿被采样，采样窗口见 [`02-electrical-and-timing.md`](02-electrical-and-timing.md) 第 4 节。

本模块的使用情况：

| 引脚 | 用途 | 说明 |
| --- | --- | --- |
| GPIO0 | BOOT 按键 | 10 kΩ 上拉，按下接地。上电或复位时被拉低则进入下载模式 |
| GPIO3 | SD 数据线 DATA3 | 10 kΩ 上拉。参与 JTAG 源选择 |
| GPIO45 | 扩展座 | 参与 VDD_SPI 电压选择。ESP32-S3R8 的 VDD_SPI 由 eFuse 固定为 3.3 V（`EFUSE_VDD_SPI_FORCE` = 1、`EFUSE_VDD_SPI_TIEH` = 1），该选择功能不生效 |
| GPIO46 | 扩展座 | 参与启动模式与 ROM 代码串口打印控制 |

约束：在 GPIO45 / GPIO46 上外接电路时，须保证其在 CHIP_PU 上升沿及其后 3 ms 内不被驱动至非预期电平。避免外接具有确定输出的驱动器。

正常启动的启动字为 `0x2b`（`SPI_FAST_FLASH_BOOT`）。

依据：ESP32-S3 数据手册 §2.7、§2.8。

## 3. 电源

| 项 | 值 | 来源 |
| --- | --- | --- |
| 供电 | 5 V，Type-C | [文档] |
| 电池 | 3.7 V 聚合锂电，1.25 mm 2P 座 | [文档] |
| 电池检测 | IO8，分压比 2（实际电压 = ADC 读数 × 2） | [文档] |

电流、充电与热的数值见 [`02-electrical-and-timing.md`](02-electrical-and-timing.md) 第 5、6 节；背光规格与调光方式见第 7 节。

## 4. 机械

| 项 | 值 |
| --- | --- |
| 模块外形 | 54.50 (W) × 101.50 (H) × 10.00 (D) mm |
| 屏外形 | 54.50 (W) × 83.00 (H) × 3.2 (D) mm，不含排线与背胶 |
| 有效显示区 | 48.96 (W) × 73.44 (H) mm |
| 有效触摸区 | 54.50 (W) × 83.00 (H) mm |
| 三维模型 | `3.5inch_ESP32-S3_board.step`（厂商资料包 `3-尺寸图`） |

### 4.1 连接器

| 接口 | 规格 |
| --- | --- |
| 显示触摸屏 FPC | 40P，0.5 mm 间距 |
| 串口 | 1.25 mm 4P |
| 扩展 IO | 1.25 mm 4P |
| I2C 扩展 | 1.25 mm 4P |
| 电池 | 1.25 mm 2P |
| 喇叭 | 1.25 mm 2P |
| 程序下载与供电 | Type-C |

随附配件：4P 1.25 mm 转 2.54 mm 接线端子线、Type-C 数据电源线。

## 5. 板上其它外设

本节各功能均未实测。

| 功能 | 器件 | 厂商示例 |
| --- | --- | --- |
| 触摸 | ST77922 内置 | `1-示例程序_Demo/Arduino/Demo/Example_15_RGB_LED_TOUCH`、`Example_28_touch_pen`；IDF 侧 `components/esp_bsp/bsp_touch.c` |
| 音频播放 / 录音 | ES8311 + 8002 系列功放 + MEMS 麦克风 | `Example_16_music`、`Example_17_echo` |
| MicroSD | SDIO 4 位 | `Example_05_show_SD_jpg_picture` |
| RGB 指示灯 | XL-5050RGBC-WS2812B | `Example_06_RGB_LED` |
| 电池电压 | IO8 ADC | `Example_13_Get_Battery_Voltage` |
| 背光 PWM | LEDC | `Example_14_Backlight_PWM` |
| 按键中断 | IO0 | `Example_09_key_interrupt` |

## 6. 引脚类勘误

厂商文件 `5-原理图_Schematic/ESP32-S3_IO资源分配表.xlsx` 存在以下错误。该表标题为 "ESP32-WROOM-32E 模组 IO 资源分配表"，与其自身内容（ESP32-S3 引脚）不符。

| # | 错误 | 正确值 |
| --- | --- | --- |
| 1 | 表标题标注模组为 ESP32-WROOM-32E | ESP32-S3R8 |
| 2 | 标注 IO48 为触摸中断输入、IO47 为触摸复位 | IO47 为中断输入，IO48 为复位（低有效） |
| 3 | 标注 IO42 为"液晶屏命令/数据选择控制引脚" | IO42 未使用；该屏为纯 QSPI 接口，无 DC 信号 |
| 4 | 标注 IO45 / IO46 未引出 | 二者引出至 1.25 mm 4P 扩展座 |
| 5 | IO43 / IO44 行的"外部设备连接说明"与其"片内默认连接说明"列（U0TXD / U0RXD）方向相反 | 见 2.5 节 |

同表的以下内容正确且可用：QSPI 五线映射、IO41 背光、IO38/IO39 I2C、IO8 电池、音频全部引脚、SD 全部引脚、IO26–IO37 被 Flash/PSRAM 占用。
