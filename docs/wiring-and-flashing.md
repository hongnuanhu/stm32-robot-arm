# 接线与烧录

## 1. 准备工具

目标芯片为 `STM32F103C8T6`，Keil 工程中的器件名称为 `STM32F103C8`。两套工程配置了 `Arm Compiler 5.06 update 5` 和 `Keil.STM32F1xx_DFP.2.4.1`。在 Windows 上先安装 ST-LINK 驱动，再准备含相应编译器的 Keil 环境。器件包可在 Keil 的 **Project → Manage → Pack Installer** 中安装，也可下载 `.pack` 后通过 **File → Import** 导入。

[ST-LINK Windows 驱动](https://www.st.com/en/development-tools/stsw-link009.html) · [STM32F1 器件包](https://www.keil.arm.com/packs/stm32f1xx_dfp-keil/versions/)

## 2. 连接 ST-LINK

断开舵机电源，断电连接主控的 SWD 接口，再给主控上电。以烧录器和主板的引脚标识为准。

| ST-LINK 信号 | STM32 引脚 |
| --- | --- |
| SWDIO | PA13 / SWDIO |
| SWCLK | PA14 / SWCLK |
| GND | GND |

若烧录器有 `VTref` 或 `VAPP` 输入，将其接到已供电主板的 3.3 V 参考端。主板的供电使用单一来源。 [ST-LINK/V2 接线手册](https://www.st.com/resource/en/user_manual/dm00026748.pdf)

## 3. 在 Keil 中编译和下载

1. 打开套件中与接线方式对应的 `.uvprojx`。建议在工程副本中编译。
2. 按 **Alt + F7** 打开工程选项。在 **Debug** 选择 **ST-Link Debugger**，再进入 **Settings**，将 **Port** 设为 **SW**。读取到 **IDCODE** 后继续。
3. 按 **F7** 编译，看到 `0 Error(s)` 后按 **F8** 下载。看到 `Verify OK` 表示写入并校验成功。将 `BOOT0` 置于 `0`，按主板的 **RESET**。

[Keil Flash 下载说明](https://www.keil.com/support/man/docs/uv4cl/uv4cl_flash_programming.htm)

## 4. 验证电脑控制

| 连接 | 接线 |
| --- | --- |
| USB-TTL → STM32 | TXD → PA10，RXD → PA9，GND 共地。使用 3.3 V 串口电平。 |
| STM32 → PCA9685 | PB6 → SCL，PB7 → SDA，3.3 V → VCC，GND 共地。 |
| PCA9685 → 舵机 | CH0–CH5 依次对应底座、肩、肘、腕部旋转、腕部俯仰、夹爪。V+ 接符合舵机规格的外置电源并共地。 |

打开配套上位机，刷新串口并选择 USB-TTL 对应的 COM 口，串口速率为 `115200`。确认界面显示 `PCA9685 OK` 后，先小幅调节一个空载关节。使用 Python 版本时，从套件上位机目录执行以下命令。

```powershell
python -m pip install -r requirements.txt
python .\robot-arm.py
```

## 5. 验证双臂示教

示教臂的五路模拟量输入使用 `PA0–PA4`。如果电位器滑动端直接接板上的 ADC 输入，其模拟电压应在 ADC 参考电压范围内。OLED 使用 `PB11`（SCL）和 `PB10`（SDA）。双臂模式的六路舵机 PWM 信号按工程通道顺序接 `PA6`、`PA7`、`PB0`、`PB1`、`PB6`、`PB7`，舵机电源与主控共地。 [STM32F103C8 数据手册](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf)

进入 OLED 的「ADC 角度读取」界面，先逐个拨动示教关节，观察执行臂是否同向、顺畅地跟随。套件固件中 `PB14` 切换夹爪状态，`PB15` 每按一次保存一组姿态，`PB13` 在保存两组及以上姿态后回放，`PB12` 退出该界面并清空记录。

## 常见现象

| 现象 | 优先检查 |
| --- | --- |
| Keil 提示找不到 Arm Compiler 5 | 检查工程选择的编译器及本机已安装版本 |
| ST-LINK 能被电脑识别，Keil 提示找不到目标 | 主板供电、GND、SWDIO 和 SWCLK |
| 上位机有串口，但 PCA9685 显示 ERR | 确认驱动板的 VCC 和 GND，再核对 SDA、SCL |
| 舵机抖动、主控复位 | 检查舵机电源、共地以及关节有无卡滞 |
