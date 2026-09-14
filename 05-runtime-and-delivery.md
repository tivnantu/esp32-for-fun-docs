# 运行时模型与交付

## 1. 运行时模型

基准实现的运行时行为：

| 项 | 说明 |
| --- | --- |
| 应用结构 | 单个 `app_main()`，顺序执行：初始化总线与面板 → 点亮背光 → 绘制 → 返回 |
| 任务 | 无自定义任务。`app_main` 返回后无应用代码运行 |
| 图像保持 | 图像存于面板 GRAM，面板自行刷新。静态画面持续显示，CPU 不参与 |
| 绘制语义 | `esp_lcd_panel_draw_bitmap()` 异步入队后返回，队列深度 10 |
| 传输完成通知 | 未注册 `on_color_trans_done` 回调，无流控 |

上表「图像保持」为观测结论：基准实现中 `app_main` 返回后画面持续显示，CPU 不参与刷新。

由此产生的约束：

- 静态内容无需周期重绘。
- 需要动态内容时须自行建立刷新循环。
- 连续绘制达队列深度后调用会阻塞。
- 需要精确节流时须注册传输完成回调。

## 2. 低功耗

未定义。deep sleep 不复位 EN，因此唤醒后面板 GRAM 内容与背光引脚电平的状态未标定。低功耗设计前须实测，见 [`07-open-items.md`](07-open-items.md)。

## 3. Flash 分区

| 项 | 实测组合 | 厂商参考 |
| --- | --- | --- |
| 分区表 | ESP-IDF 默认（单 factory 应用，1 MB） | 自定义 |
| 应用镜像 | 254,224 B（余量 76%） | — |
| OTA | 未配置 | 按自建分区表 |

1 MB 应用分区对纯显示应用余量充足。引入 GUI 框架、字库或图片资源后须重新评估；此类资源宜置于独立只读分区或 MicroSD 卡。

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

未启用 `CONFIG_APP_REPRODUCIBLE_BUILD` 时，镜像内嵌构建时间戳，重新构建的产物 SHA-256 不同；该值仅标识本次产物，不作为版本判据。

烧录后可用 `esptool verify_flash` 在设备侧核对已写入的镜像。

## 6. 第三方组件与许可证

| 组件 | 许可证 |
| --- | --- |
| espressif/esp_lcd_st77922 | Apache-2.0 |
| espressif/cmake_utilities | Apache-2.0 |
| ESP-IDF | Apache-2.0；其中随附的第三方组件各自适用其自带许可证 |
| 8×8 位图字体 | Public Domain |

再分发前须逐项核对许可证条款。厂商资料包内示例代码的许可条款以其文件内声明为准。
