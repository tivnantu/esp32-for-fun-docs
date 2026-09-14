# 编译与烧录

本章命令以 Linux（Arch Linux、x86_64）为基准。设备节点、串口权限组与 EIM 获取方式随平台变化，见第 7 节与第 9 节。

## 1. 版本选择依据

| 约束 | 值 |
| --- | --- |
| `esp_lcd_st77922` 2.0.2 要求 | ESP-IDF `>= 5.4` |
| `esp_lcd_st77922` 1.0.3 要求 | ESP-IDF `> 5.0.4, != 5.1.1` |
| 实测组合 | ESP-IDF 5.5.4 + `esp_lcd_st77922` 2.0.2 |
| 厂商验证组合 | ESP-IDF 5.4.2 + `esp_lcd_st77922` 1.0.3 + LVGL 8.4.0 |

ESP-IDF 6.x 未经验证。组件 2.0.2 的声明要求为 IDF `>=5.4`，未声明对 6.x 的支持。`bits_per_pixel` 等字段在 6.0 与 6.1 的头文件中仍存在。

[勘误] 厂商 `1-示例程序_Demo/ESP-IDF/3.5inch_ESP32-S3_LVGL/main/idf_component.yml` 声明 `idf: ">=5.1"`，与其锁定的 1.0.3 及 IDF 5.4.2 不一致。以组件自身声明为准。

## 2. 安装 ESP-IDF

### 2.1 官方安装器（EIM）

```
eim install -i v5.5.4 -t esp32s3 -n true -p ~/esp
```

`-t esp32s3` 限定只安装该目标的工具链。安装内容：ESP-IDF 源码至 `~/esp/v5.5.4/esp-idf`，工具链至 `~/.espressif/tools`。

EIM 的获取方式按平台而定（Arch：`paru -S eim-cli`）。

### 2.2 传统方式

```
git clone -b v5.5.4 --recursive https://github.com/espressif/esp-idf.git ~/esp/v5.5.4/esp-idf
cd ~/esp/v5.5.4/esp-idf && ./install.sh esp32s3
```

## 3. 激活环境

### 3.1 EIM 安装

```
source ~/.espressif/tools/activate_idf_v5.5.4.sh
```

该脚本的 `is_sourced()` 以 `${0##*/}` 是否属于 `dash|bash|ksh|sh` 判定。`$0` 为空或为其他值时，脚本判定为「被执行」并退出。交互式终端下正常；在 `$0` 为空的非交互环境中须显式指定：

```
bash -c '. ~/.espressif/tools/activate_idf_v5.5.4.sh && idf.py build' bash
```

### 3.2 传统方式安装

```
source ~/esp/v5.5.4/esp-idf/export.sh
```

### 3.3 两种布局不可混用

EIM 将虚拟环境置于 `~/.espressif/tools/python/<IDF 版本>/venv`；ESP-IDF 自带的 `export.sh` 期望 `python_env/` 目录布局。在 EIM 布局下 `export.sh` 不可直接使用，须同时设置 `IDF_PYTHON_ENV_PATH`，或改用 3.1 节的激活脚本。

## 4. 最小工程骨架

```
<project>/
  CMakeLists.txt             顶层，project(<name>)
  sdkconfig.defaults         目标、Flash、控制台
  main/
    CMakeLists.txt
    idf_component.yml        组件依赖
    main.c                   应用
```

`main/idf_component.yml`：

```yaml
dependencies:
  idf: ">=5.4"
  espressif/esp_lcd_st77922: "^2.0.2"
```

`sdkconfig.defaults`：

```
CONFIG_IDF_TARGET="esp32s3"
CONFIG_ESPTOOLPY_FLASHSIZE_16MB=y
CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG=y
```

可选，若需使用 8 MB PSRAM 与 240 MHz 主频（厂商参考配置）：

```
CONFIG_SPIRAM=y
CONFIG_SPIRAM_MODE_OCT=y
CONFIG_SPIRAM_SPEED_80M=y
CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ_240=y
```

`CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG=y` 决定日志是否出现在 Type-C 对应的串口上。缺省为 UART0，日志只通过 IO43/IO44 输出。

## 5. 编译

```
idf.py set-target esp32s3
idf.py build
```

首次 `set-target` 会解析 `idf_component.yml` 并下载组件至 `managed_components/`。`dependencies.lock` 记录组件解析结果，应纳入版本控制。

该锁文件只锁定组件（版本与 `component_hash`），**不锁定 ESP-IDF**：其中的 `idf` 条目是约束区间，构建时组件管理器以当前 IDF 版本覆盖该条目。复现实测组合须安装第 1 节指定的 IDF 版本本身，不得以「满足组件声明区间」代替。

## 6. 烧录

Type-C 口对应 ESP32-S3 的原生 USB-Serial-JTAG。

```
python -m esptool --chip esp32s3 -b 460800 \
  --before default_reset --after hard_reset -p /dev/ttyACM0 \
  write_flash --flash_mode dio --flash_size 16MB --flash_freq 80m \
  0x0 build/bootloader/bootloader.bin \
  0x8000 build/partition_table/partition-table.bin \
  0x10000 build/<project>.bin
```

偏移量为 IDF 默认单 factory 分区布局。`<project>` 为工程名，与应用镜像文件名一致。`--flash_mode` 取 `dio` 或 `qio` 均可启动。

等价的 IDF 命令：

```
idf.py -p /dev/ttyACM0 flash
```

## 7. 串口

| 项 | 值 |
| --- | --- |
| 接口类型 | USB-Serial-JTAG（芯片原生，非板载 USB 转串口芯片） |
| USB ID | `303a:1001`，Espressif USB JTAG/serial debug unit |
| 驱动 | CDC-ACM，Linux 免驱 |
| 设备节点 | `/dev/ttyACM0`（Linux） |
| 权限组 | Arch: `uucp`；Debian/Ubuntu: `dialout` |
| 波特率 | 该接口为 USB CDC，波特率设置不影响实际传输速率 |

加入权限组后需重新登录生效：

```
sudo usermod -aG uucp $USER
```

板载另有 1.25 mm 4P 串口座，引出 UART0（IO43 / IO44），可外接 USB 转串口模块作为替代通道。

## 8. 下载模式

通常由 esptool 自动复位进入。自动进入失败时手动操作：

- 按住 BOOT（IO0），上电或短按 RESET，随后松开 BOOT
- 或：上电状态下按住 BOOT，短按 RESET，松开 RESET，随后松开 BOOT

## 9. 已知环境约束

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| `source activate_idf_*.sh` 报 "should be sourced, not executed" | 见第 3 节 | 在交互终端执行，或显式指定 `bash` |
| `export.sh` 报找不到 `python_env/` 下的虚拟环境 | EIM 虚拟环境路径不同 | 使用 `activate_idf_*.sh` |
| 烧录报权限拒绝 | 当前用户不在串口权限组 | 加入 `uucp` / `dialout` 并重新登录 |
| `idf.py monitor` 无输出 | 控制台未切至 USB-Serial-JTAG | 置 `CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG=y` |
