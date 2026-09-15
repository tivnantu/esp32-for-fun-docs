# 运行时模型与交付

## 1. 运行时模型

基准实现的运行时行为：

| 项 | 说明 |
| --- | --- |
| 应用结构 | 单个 `app_main()`：初始化总线与面板 → 点亮背光 → 打印场景列表 → 进入主循环 |
| 任务 | 主循环位于 `app_main` 内；另有一个控制台任务读取串口命令。无其他自定义任务 |
| 图像保持 | 图像存于面板 GRAM，面板自行刷新。停止重绘后画面持续显示，CPU 不参与 |
| 绘制语义 | `esp_lcd_panel_draw_bitmap()` 先排空在途传输，再下发本带的 `CASET`、`RASET`、`RAMWR`；返回时上一带已传完、本带已入队，有效重叠 1 带 |
| 传输完成通知 | 未注册 `on_color_trans_done` 回调，无流控 |

上表「图像保持」为观测结论：停止重绘后画面持续显示，CPU 不参与刷新。

由此产生的约束：

- 静态内容无需周期重绘。
- 需要动态内容时须在 `app_main` 的主循环内自行建立刷新节奏。
- 需要精确节流时须注册传输完成回调。

## 2. 低功耗

未定义。deep sleep 不复位 EN，因此唤醒后面板 GRAM 内容与背光引脚电平的状态未标定。低功耗设计前须实测，见 [`07-open-items.md`](07-open-items.md)。

## 3. Flash 分区

| 项 | 实测组合 | 厂商参考 |
| --- | --- | --- |
| 分区表 | ESP-IDF 默认（单 factory 应用，1 MB） | 自定义 |
| 应用镜像 | 284,208 B（余量 73%）；可选子系统全开 461,872 B（余量 56%） | — |
| OTA | 未配置 | 按自建分区表 |

可选子系统指触摸、RGB 指示灯、音频、电池计与 MicroSD，缺省全部关闭。两种配置均为 CPU 160 MHz、无 GUI 框架，差额来自这五项的驱动与场景代码。

1 MB 应用分区对纯显示应用余量充足；可选子系统全开后余量降至 56%。引入 GUI 框架、字库或图片资源后须重新评估；此类资源宜置于独立只读分区或 MicroSD 卡。

## 4. 内存预算

| 项 | 大小 | 位置 |
| --- | --- | --- |
| 分带缓冲 | 20 KB（320 × 32 × 2 B） | 静态分配 |
| 文本块缓冲 | 12.7 KB（264 × 24 × 2 B） | 堆，DMA 能力 |
| OPI PSRAM | 8 MB，未启用 | — |

启用 OPI PSRAM 的配置见 [`01-hardware.md`](01-hardware.md) 第 1 节。

## 5. 交付物标识

每次交付须记录：

| 项 | 来源 |
| --- | --- |
| 应用版本 | 仓库须有 `vMAJOR.MINOR.PATCH` 形式的标签，取 `git describe --tags --dirty --always` 的输出，并与镜像内 `esp_app_desc_t.version` 一致 |
| 源码标识 | 上述输出对应的提交哈希（`git rev-parse HEAD`）。构建前 `git status --porcelain` 须为空，否则本项无效 |
| 应用镜像校验和 | 对 `build/bootloader/bootloader.bin`、`build/partition_table/partition-table.bin`、`build/<project>.bin` **分别**计算 SHA-256 |
| 工具链版本 | ESP-IDF 与组件版本，见 `README.md`「版本基准」 |

未启用 `CONFIG_APP_REPRODUCIBLE_BUILD` 时，镜像内嵌构建时间戳，重新构建的产物 SHA-256 不同；该值仅标识该次构建产物，不作为版本判据。

烧录后可用 `esptool verify_flash` 在设备侧核对已写入的镜像。

## 6. 第三方组件与许可证

| 组件 | 许可证 |
| --- | --- |
| espressif/esp_lcd_st77922 | Apache-2.0 |
| espressif/cmake_utilities | Apache-2.0 |
| espressif/esp_codec_dev | Apache-2.0 |
| espressif/led_strip | Apache-2.0 |
| ESP-IDF | Apache-2.0；其中随附的第三方组件各自适用其自带许可证 |
| 8×8 位图字体 | Public Domain |

再分发前须逐项核对许可证条款。厂商资料包内示例代码的许可条款以其文件内声明为准。
