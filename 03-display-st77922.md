# ST77922 · QSPI 上屏

## 1. 屏参数

| 项 | 值 | 来源 |
| --- | --- | --- |
| 尺寸 / 类型 | 3.5" IPS TFT | [文档] |
| 分辨率 | 320 (H) × 480 (V) | [实测+交叉] |
| 驱动 IC | ST77922 | [实测+交叉] |
| 接口 | QSPI | [实测+交叉] |
| 色彩深度 | 16 bpp（RGB565）常用；最高 24 bpp | [文档] |
| 有效显示区 | 48.96 (W) × 73.44 (H) mm | [文档] |
| 可视角度 | ALL | [文档] |

背光规格见 [`02-electrical-and-timing.md`](02-electrical-and-timing.md) 第 7 节。

## 2. 硬件连接

| 信号 | GPIO |
| --- | --- |
| CS | IO10 |
| PCLK | IO12 |
| D0 | IO11 |
| D1 | IO13 |
| D2 | IO14 |
| D3 | IO9 |
| RST | 与 ESP32-S3 EN 共用，无独立 GPIO |
| BL | IO41，高有效 |

两条硬件约束：

1. **复位不可软件控制。** 面板复位线接至主控 EN，两者同源。`esp_lcd_panel_dev_config_t.reset_gpio_num` 必须为 `-1`，复位由驱动下发软件复位命令完成。
2. **无 DC 信号。** 命令与数据的区分完全由 QSPI 操作码承担，不存在命令/数据选择引脚。

面板侧接口模式由 IM2 / IM1 / IM0 选择，QSPI 对应 `IM2 = 0`、`IM1 = 1`、`IM0 = 1`（数据手册 §5.2）。本模块按此接法。

## 3. QSPI 协议

ST77922 数据手册 §7.8.5.1、§7.8.5.2（V1.2 编号；V0.1 中对应 §8.8.5.1、§8.8.5.2）。

### 3.1 命令与参数写入

主机先发 1 字节写指令，取值 `0x02`、`0xA2`、`0x32`、`0x38`；随后发 3 字节 `AD[23:0]`，其组成为 1 字节 `0x00`、1 字节命令地址、1 字节 `0x00`；其后字节为参数。

### 3.2 读出

主机先发 `0x0B`，随后发 3 字节 `AD[23:0]`（同样为 `0x00` / 命令地址 / `0x00`）；其后由面板输出参数。数据手册的 QSPI 读时序另标注有 dummy 周期。读出路径未实现，且未验证。

### 3.3 像素写入

使用操作码 `0x32`，命令地址为 `0x2C`（`RAMWR`）。

### 3.4 esp_lcd 侧的编码

`esp_lcd_st77922` 在 QSPI 模式下把 32 位命令字按下列方式组装，与上述帧格式一致：

```
lcd_cmd = (opcode << 24) | (0x00 << 16) | (cmd << 8) | 0x00
```

| 项 | 值 |
| --- | --- |
| 参数/命令操作码 | `0x02` |
| 像素操作码 | `0x32` |
| 读操作码 | `0x0B` |

面板 IO 配置由 `ST77922_PANEL_IO_QSPI_CONFIG(cs, cb, ctx)` 给出：`dc_gpio_num = -1`、`lcd_cmd_bits = 32`、`lcd_param_bits = 8`、`flags.quad_mode = 1`、`trans_queue_depth = 10`。

`pclk_hz` 宏默认值为 40 MHz，必须按第 5 节覆盖。

## 4. 初始化序列

### 4.1 驱动内置序列不适用

`espressif/esp_lcd_st77922` 2.0.2 的 `vendor_specific_init_default` 面向 532×300 面板，其特征值为：

```
CASET  0x0000 – 0x0213     532 列
RASET  0x0000 – 0x012B     300 行
```

本模块为 320×480。使用内置序列的后果：面板正常供电、背光可点亮、所有 `esp_lcd` 调用返回 `ESP_OK`，但无任何显示。此类故障不可由 API 返回值判别。

### 4.2 本模块序列

分页命令：`0xF1` 切 CMD2，`0xF2` 切 CMD3，`0xF0` 切回 CMD1。`CASET`、`RASET`、`SLPOUT`、`DISPON`、`COLMOD`、`MADCTL` 均在 CMD1 内下发。

```c
static const st77922_lcd_init_cmd_t lcd_init_cmds[] = {
    {0xF1, (uint8_t[]){0x00}, 1, 0},
    {0x60, (uint8_t[]){0x00, 0x00, 0x00}, 3, 0},
    {0x65, (uint8_t[]){0x80}, 1, 0},
    {0x79, (uint8_t[]){0x06}, 1, 0},
    {0x7B, (uint8_t[]){0x00, 0x08, 0x08}, 3, 0},
    {0x80, (uint8_t[]){0x55, 0x62, 0x2F, 0x17, 0xF0, 0x52, 0x70, 0xD2, 0x52, 0x62, 0xEA}, 11, 0},
    {0x81, (uint8_t[]){0x26, 0x52, 0x72, 0x27}, 4, 0},
    {0x84, (uint8_t[]){0x92, 0x25}, 2, 0},
    {0x87, (uint8_t[]){0x10, 0x10, 0x58, 0x00, 0x02, 0x3A}, 6, 0},
    {0x88, (uint8_t[]){0x00, 0x00, 0x2C, 0x10, 0x04, 0x00, 0x00, 0x00, 0x01, 0x01, 0x01, 0x01, 0x01, 0x00, 0x06}, 15, 0},
    {0x89, (uint8_t[]){0x00, 0x00, 0x00}, 3, 0},
    {0x8A, (uint8_t[]){0x13, 0x00, 0x2C, 0x00, 0x00, 0x2C, 0x10, 0x10, 0x00, 0x3E, 0x19}, 11, 0},
    {0x8B, (uint8_t[]){0x15, 0xB1, 0xB1, 0x44, 0x96, 0x2C, 0x10, 0x97, 0x8E}, 9, 0},
    {0x8C, (uint8_t[]){0x1D, 0xB1, 0xB1, 0x44, 0x96, 0x2C, 0x10, 0x50, 0x0F, 0x01, 0xC5, 0x12, 0x09}, 13, 0},
    {0x8D, (uint8_t[]){0x0C}, 1, 0},
    {0x8E, (uint8_t[]){0x33, 0x01, 0x0C, 0x13, 0x01, 0x01}, 6, 0},
    {0xB3, (uint8_t[]){0x00, 0x30}, 2, 0},
    {0xF1, (uint8_t[]){0x00}, 1, 0},
    {0x71, (uint8_t[]){0xD0}, 1, 0},
    {0x66, (uint8_t[]){0x02, 0x3F}, 2, 0},
    {0xBE, (uint8_t[]){0x26, 0x00, 0x9D}, 3, 0},
    {0x70, (uint8_t[]){0x01, 0xA0, 0x11, 0x40, 0xE0, 0x00, 0x11, 0x69, 0x11, 0x00, 0x00, 0x1A}, 12, 0},
    {0x90, (uint8_t[]){0x04, 0x04, 0x55, 0x74, 0x00, 0x40, 0x43, 0x27, 0x27}, 9, 0},
    {0x91, (uint8_t[]){0x04, 0x04, 0x55, 0x75, 0x00, 0x40, 0x42, 0x27, 0x27}, 9, 0},
    {0x92, (uint8_t[]){0x04, 0x44, 0x55, 0xC0, 0x06, 0x00, 0x07, 0x05, 0x90, 0x27}, 10, 0},
    {0x93, (uint8_t[]){0x04, 0x43, 0x11, 0x00, 0x00, 0x00, 0x00, 0x05, 0x90, 0x27}, 10, 0},
    {0x94, (uint8_t[]){0x00, 0x00, 0x00, 0x00, 0x00, 0x00}, 6, 0},
    {0x95, (uint8_t[]){0x96, 0x16, 0x00, 0x00, 0xFF}, 5, 0},
    {0x96, (uint8_t[]){0x44, 0x53, 0x03, 0x12, 0x23, 0x24, 0x06, 0x05, 0x94, 0x27, 0x00, 0x44}, 12, 0},
    {0x97, (uint8_t[]){0x44, 0x53, 0x47, 0x56, 0x20, 0x20, 0x02, 0x01, 0x94, 0x27, 0x00, 0x44}, 12, 0},
    {0xBA, (uint8_t[]){0x55, 0x94, 0x2D, 0x94, 0x27}, 5, 0},
    {0x9A, (uint8_t[]){0x40, 0x00, 0x06, 0x00, 0x00, 0x00, 0x00}, 7, 0},
    {0x9B, (uint8_t[]){0x00, 0x00, 0x06, 0x00, 0x00, 0x00, 0x00}, 7, 0},
    {0x9C, (uint8_t[]){0x5C, 0x12, 0x00, 0x00, 0x10, 0x12, 0x00, 0x00, 0x10, 0x02, 0x00, 0x00, 0x00}, 13, 0},
    {0x9D, (uint8_t[]){0x8A, 0x51, 0x00, 0x00, 0x00, 0x80, 0x1E, 0x01}, 8, 0},
    {0x9E, (uint8_t[]){0x51, 0x00, 0x00, 0x00, 0x80, 0x1E, 0x01}, 7, 0},
    {0xB4, (uint8_t[]){0x1D, 0x1C, 0x1E, 0x0B, 0x14, 0x02, 0x13, 0x09, 0x1E, 0x00, 0x1E, 0x10}, 12, 0},
    {0xB5, (uint8_t[]){0x1D, 0x1C, 0x1E, 0x0A, 0x15, 0x03, 0x11, 0x08, 0x1E, 0x01, 0x1E, 0x12}, 12, 0},
    {0xB6, (uint8_t[]){0x77, 0x77, 0x00, 0x0A, 0xFF, 0x0A, 0xFF}, 7, 0},
    {0x86, (uint8_t[]){0xCD, 0x04, 0xB1, 0x02, 0x58, 0x12, 0x58, 0x0C, 0x13, 0x01, 0xA5, 0x00, 0xA5, 0xA5}, 14, 0},
    {0xB7, (uint8_t[]){0x07, 0x0A, 0x0E, 0x06, 0x05, 0x03, 0x2B, 0x03, 0x03, 0x42, 0x07, 0x10, 0x10, 0x2E, 0x3F, 0x0D}, 16, 0},
    {0xB8, (uint8_t[]){0x07, 0x0A, 0x0D, 0x05, 0x05, 0x02, 0x2B, 0x02, 0x03, 0x42, 0x06, 0x10, 0x0F, 0x2E, 0x3F, 0x0D}, 16, 0},
    {0xB9, (uint8_t[]){0x23, 0x23}, 2, 0},
    {0xBF, (uint8_t[]){0x10, 0x14, 0x14, 0x0B, 0x0B, 0x0B}, 6, 0},
    {0xF2, (uint8_t[]){0x00}, 1, 0},
    {0x73, (uint8_t[]){0x04, 0xDA, 0x12, 0x54, 0x47}, 5, 0},
    {0x77, (uint8_t[]){0x6B, 0x5B, 0xFD, 0xC3, 0xC5}, 5, 0},
    {0x7A, (uint8_t[]){0x15, 0x27}, 2, 0},
    {0x7B, (uint8_t[]){0x04, 0x57}, 2, 0},
    {0x7E, (uint8_t[]){0x01, 0x0E}, 2, 0},
    {0xBF, (uint8_t[]){0x36}, 1, 0},
    {0xE3, (uint8_t[]){0x40, 0x40}, 2, 0},
    {0xF0, (uint8_t[]){0x00}, 1, 0},
    {0xD0, (uint8_t[]){0x00}, 1, 0},
    {0x2A, (uint8_t[]){0x00, 0x00, 0x01, 0x3F}, 4, 0},   /* CASET 0..319 */
    {0x2B, (uint8_t[]){0x00, 0x00, 0x01, 0xDF}, 4, 0},   /* RASET 0..479 */
    {0x21, (uint8_t[]){0x00}, 0, 0},                     /* INVON */
    {0x11, (uint8_t[]){0x00}, 0, 120},                   /* SLPOUT */
    {0x29, (uint8_t[]){0x00}, 0, 0},                     /* DISPON */
    {0x2C, (uint8_t[]){0x00}, 0, 0},                     /* RAMWR */
    {0x3A, (uint8_t[]){0x01}, 1, 0},                     /* COLMOD 16bpp */
    {0x36, (uint8_t[]){0x00}, 1, 0},                     /* MADCTL */
    {0x35, (uint8_t[]){0x01}, 1, 20},                    /* TEON */
};
```

共 63 条。该序列与厂商 Arduino 驱动（`1-示例程序_Demo/Arduino/Install libraries/ST77922/ST77922.cpp`）及厂商 ESP-IDF BSP（`1-示例程序_Demo/ESP-IDF/3.5inch_ESP32-S3_LVGL/components/esp_bsp/bsp_display.c`）一致；比对方式为逐条比较命令字、参数字节数与延时三元组。

序列已包含 `INVON`（`0x21`）与 `COLMOD = 16 bpp`（`0x3A = 0x01`）。`esp_lcd_panel_init()` 会先下发一次 `MADCTL` 与 `COLMOD`，被本序列覆盖，驱动会输出两条覆盖警告，属预期。

## 5. 必需配置

| 项 | 值 | 依据 |
| --- | --- | --- |
| SPI 主机 | `SPI2_HOST` | [实测+交叉] |
| `pclk_hz` | 80 MHz（高于数据手册写时序上限，见 5.1 节） | [实测+交叉] |
| `reset_gpio_num` | `-1` | [实测+交叉] |
| `bits_per_pixel` | 16 | [实测+交叉] |
| `rgb_ele_order` | `LCD_RGB_ELEMENT_ORDER_RGB` | [实测+交叉] |
| `vendor_config.flags.use_qspi_interface` | 1 | [实测+交叉] |
| `vendor_config.init_cmds` | 第 4.2 节序列 | [实测+交叉] |
| `data_endian` | 不适用，见第 6 节 | [实测+交叉] |
| `max_transfer_sz`（总线配置） | 不得小于单次传输的最大字节数。基准实现取 `320 × 80 × 2 = 51,200 B` | [实测+交叉] |

`ST77922_PANEL_IO_QSPI_CONFIG` 宏的 `pclk_hz` 默认值为 40 MHz，必须显式覆盖为 80 MHz。

`max_transfer_sz` 未显式设置时按驱动默认值取值，可能小于一次分带传输的数据量；此时该次 `esp_lcd_panel_draw_bitmap()` 会返回 `ESP_ERR_INVALID_ARG`。

### 5.1 时钟频率与数据手册上限

ST77922 数据手册 V1.2 表 5（QSPI Interface Characteristics）对写模式规定：

| 参数 | 符号 | 最小值 |
| --- | --- | --- |
| 串行时钟周期（写） | `TSCYCW` | 16 ns |
| 时钟高电平宽度（写） | `TSHW` | 7 ns |
| 时钟低电平宽度（写） | `TSLW` | 7 ns |

由此得出写时钟上限 62.5 MHz。厂商参考配置与实测均使用 80 MHz（周期 12.5 ns，高低电平各 6.25 ns），低于上述最小值。

厂商依据：Arduino 驱动 `ST77922.h` 定义 `QSPI_FREQUENCY 80000000`；ESP-IDF BSP 定义 `EXAMPLE_LCD_PIXEL_CLOCK_HZ (80 * 1000 * 1000)`。两者与实测配置一致，在本模块上工作正常。

如需严格满足数据手册限值，将 `pclk_hz` 降至 62.5 MHz 以下。降频后的行为未验证。

## 6. RGB565 字节序

### 6.1 结论

写入 `esp_lcd_panel_draw_bitmap()` 的每个 16 位像素必须**高低字节交换**。驱动与配置层均不提供该处理。

```c
static inline uint16_t to_panel(uint16_t rgb565)
{
    return (uint16_t)((rgb565 >> 8) | (rgb565 << 8));
}
```

### 6.2 原因

面板按高字节先接收 RGB565 像素。ESP32-S3 为小端序，`uint16_t` 缓冲区经 SPI DMA 时按内存序输出低字节在前。二者相反。

以深蓝 `RGB565(0x10, 0x18, 0x40) = 0x10C8` 为例。RGB565 的分量位宽为 R 5 位、G 6 位、B 5 位，下表以各自满量程为分母：

| | 字节流 | 面板解析值 | R | G | B | 呈现 |
| --- | --- | --- | --- | --- | --- | --- |
| 不交换 | `C8 10` | `0xC810` | 25/31 | 0 | 16/31 | 品红 |
| 交换 | `10 C8` | `0x10C8` | 2/31 | 6/63 | 8/31 | 深蓝 |

白色 `0xFFFF` 在两种情况下均不变，因此该缺陷仅表现为彩色偏移，灰度与高亮内容不可见。

### 6.3 不可用的替代方案

`esp_lcd_panel_dev_config_t.data_endian` 对 ST77922 驱动无效。该字段在 `esp_lcd` 中仅由 `esp_lcd_panel_st7789.c` 读取并映射到 RAMCTL 位，`esp_lcd_st77922_general.c` 不引用。

### 6.4 厂商依据

厂商 ESP-IDF BSP 通过 LVGL 层达成等效结果：`sdkconfig.defaults` 中 `CONFIG_LV_COLOR_16_SWAP=y`，LVGL 输出的即为字节交换后的 RGB565，随后未经二次处理直接送入 `esp_lcd_panel_draw_bitmap()`。

## 7. 列坐标 4 像素对齐

ST77922 为双栅极驱动，像素数须为 4 的倍数（数据手册 §2.1）。`esp_lcd_panel_draw_bitmap()` 的 `x_start` 与 `x_end` 须各自为 4 的倍数，否则该次传输的列地址无效。

判据：区域宽度边界落在 4 的倍数上。例如宽度 320 的整屏刷新满足条件；居中放置宽度 264 的文本块时，`x_start = 28`、`x_end = 292`，二者均满足。

## 8. 最小实现

顺序：

1. `spi_bus_initialize(SPI2_HOST, &buscfg, SPI_DMA_CH_AUTO)`，`buscfg` 由 `ST77922_PANEL_BUS_QSPI_CONFIG(pclk, d0, d1, d2, d3, max_transfer_sz)` 生成。
2. `esp_lcd_new_panel_io_spi()`，配置由 `ST77922_PANEL_IO_QSPI_CONFIG(cs, NULL, NULL)` 生成，随后覆盖 `pclk_hz`。
3. `esp_lcd_new_panel_st77922()`，`vendor_config` 填入第 4.2 节序列并置 `use_qspi_interface = 1`。
4. `esp_lcd_panel_reset()` → `esp_lcd_panel_init()` → `esp_lcd_panel_invert_color(panel, true)` → `esp_lcd_panel_disp_on_off(panel, true)`。
5. 置 IO41 为高，点亮背光。
6. `esp_lcd_panel_draw_bitmap()` 写入像素。

`esp_lcd_panel_invert_color(panel, true)` 与序列中的 `0x21` 重复，幂等。保留该调用可使 IPS 反色要求显式化，不依赖序列中的单条命令。传入 `false` 会下发 `0x20`（`INVOFF`）并撤销反色。

## 9. 横屏

`esp_lcd_panel_swap_xy()` 在本驱动返回 `ESP_ERR_NOT_SUPPORTED`，无硬件换轴。横屏显示须在软件层完成，例如 LVGL 的 `LV_DISP_ROT_90` 配合刷新回调内的缓冲区转置。厂商示例即采用此方式。

## 10. 验证基线

上屏成功后，串口应输出等效于下列内容（`TAG` 随实现变化）：

```
I (111) st77922: version: 2.0.2
I (111) st77922_general: LCD panel create success, version: 2.0.2
W (351) st77922_general: The 3Ah command has been used and will be overwritten by external initialization sequence
W (351) st77922_general: The 36h command has been used and will be overwritten by external initialization sequence
I (381) hello: text box: x 28..292, y 228..252
I (381) hello: done
```

两条 `W` 由第 4.2 节的 `COLMOD` 与 `MADCTL` 覆盖内置值引起，属预期。`esp_lcd` 全部调用返回 `ESP_OK` 不构成上屏成立的证据，见第 4.1 节。
