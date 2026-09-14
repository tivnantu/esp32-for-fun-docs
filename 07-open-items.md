# 未验证项与验收

## 1. 未验证项登记

| # | 功能 | 现状 | 已知信息 | 风险 | 参考 |
| --- | --- | --- | --- | --- | --- |
| 1 | 触摸 | 未实测 | 控制器集成于 ST77922；I2C 地址 0x55；INT = IO47（低有效）；RST = IO48（低有效）；与 ES8311 共用 I2C | 与音频共用总线，初始化顺序与总线仲裁未知 | `Example_15_RGB_LED_TOUCH`、`Example_28_touch_pen`、`bsp_touch.c` |
| 2 | 音频 | 未实测 | ES8311（I2C 0x18）+ 8002 系列功放（型号标注冲突，见 [`08-reference-index.md`](08-reference-index.md) 第 4 节第 6 条）；I2S 引脚见 [`01-hardware.md`](01-hardware.md) 2.4；功放使能 IO1 低有效 | I2S 主从关系、采样率、位深、编解码寄存器配置均未确定 | `Example_16_music`、`Example_17_echo` |
| 3 | MicroSD | 未实测 | SDIO 4 位，引脚见 [`01-hardware.md`](01-hardware.md) 2.3 | DATA3 落在 strapping 引脚 GPIO3 上 | `Example_05_show_SD_jpg_picture` |
| 4 | RGB 指示灯 | 未实测 | XL-5050RGBC-WS2812B，单线，IO40 | 数据速率与复位窗口未确认 | `Example_06_RGB_LED` |
| 5 | 电池计 | 未实测 | IO8 ADC，分压比 2 | ADC 标定与量程未做 | `Example_13_Get_Battery_Voltage` |
| 6 | 背光 PWM 调光 | 未实测 | LEDC 10 位 / 5 kHz（厂商做法） | 亮度与占空比对应关系未标定；低占空比可能闪烁 | `Example_14_Backlight_PWM` |
| 7 | OPI PSRAM | 未启用 | 8 MB，启用符号见 [`01-hardware.md`](01-hardware.md) 第 1 节 | 启用后的启动时间、缓存一致性、DMA 约束未实测 | 厂商 `sdkconfig.defaults` |
| 8 | 低功耗 | 未定义 | deep sleep 不复位 EN | 唤醒后面板 GRAM 与背光状态未知 | 需实测 |
| 9 | 峰值电流 | 未测量 | 仅有厂商点值，见 [`02-electrical-and-timing.md`](02-electrical-and-timing.md) 第 5 节 | 供电端与线材选型无峰值依据 | 需电流探针测量 |
| 10 | 发热 | 未测量 | 厂商给出器件温度上限，见 [`02-electrical-and-timing.md`](02-electrical-and-timing.md) 第 6 节 | 长时间满载温升未知 | 需满载温升测试 |
| 11 | 持续刷新率 | 未测量 | 整屏填充入队耗时 ≤ 10 ms | 无持续刷新场景下的实测帧率 | 需建立刷新循环测量 |
| 12 | 横屏 | 未实测 | `esp_lcd_panel_swap_xy()` 返回 `ESP_ERR_NOT_SUPPORTED`，须软件旋转 | 软件旋转的额外开销与正确性未验证 | 厂商示例 `LV_DISP_ROT_90` |
| 13 | 器件型号取值 | 取自原理图标注 | 屏组件 `HMX035CTFT-001`；功放两处标注冲突 | 采购与 BOM 可能存在偏差 | 正式 BOM 须向厂商索取 |
| 14 | QSPI 时钟 | 80 MHz，高于数据手册写时序上限 | 数据手册 V1.2 表 5 规定 `TSCYCW` ≥ 16 ns（≈62.5 MHz）；厂商参考配置同为 80 MHz 且工作正常 | 长期可靠性未评估；温度或批次变化下可能不稳 | 严格合规须降至 ≤62.5 MHz 并重测 |

## 2. 批次风险

初始化序列针对当前玻璃标定。面板批次变更时，下列参数是否需要重新标定未知：

- `CASET` / `RASET` 边界
- `TIMINGCTRL`（`0x70`）参数
- `VGH` / `VGL` 泵设置
- Gamma 曲线

比对基准：[`03-display-st77922.md`](03-display-st77922.md) 第 4.2 节的序列，与厂商 Arduino 驱动及 ESP-IDF BSP 逐字节一致。

## 3. 验收判据

本节判据仅覆盖显示与启动。第 1 节登记的 14 项未验证功能不在本次验收范围内。

| 判据 | 方法 | 界值 |
| --- | --- | --- |
| 上屏成立 | 屏幕实际输出 | 显示内容与写入 `esp_lcd_panel_draw_bitmap()` 的像素数据一致（颜色、位置） |
| 反色与色彩正确 | 屏幕实际输出 | 写入已知单色（纯白、纯红）时屏幕呈现对应颜色 |
| 屏初始化耗时 | 串口日志时间戳 | ≤ 300 ms（基准实现） |
| 整屏填充耗时 | 串口日志时间戳 | ≤ 15 ms（基准实现） |
| 启动完成 | 串口日志 | `app_main` 正常返回 |

不可作为判据的现象：`esp_lcd_*` 系列返回值、背光点亮、`3Ah` / `36h` 覆盖警告。见 [`06-troubleshooting.md`](06-troubleshooting.md) 第 8 节。
