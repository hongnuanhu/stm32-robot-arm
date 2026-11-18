# 接线与调试笔记

这份笔记对应仓库首页视频里的 STM32 双臂机械臂。示教臂提供关节位置，执行臂跟随动作。

## 主控和舵机信号

| 模块 | STM32F103C8T6 连接 |
| --- | --- |
| ST-LINK 烧录器 | SWDIO → PA13，SWCLK → PA14，GND 共地 |
| 示教臂电位器 | 五路模拟信号接 PA0–PA4 |
| 执行臂舵机 | 六路 PWM 依次接 PA6、PA7、PB0、PB1、PB6、PB7 |
| OLED | SCL 接 PB11，SDA 接 PB10 |

电位器的滑动端直接接 ADC 时，要把信号电压限制在 ADC 参考电压范围内。舵机使用符合其规格的外置电源，并和主控共地。接线和烧录时先断开舵机电源。 [STM32F103C8 数据手册](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf)

## 主控烧录

1. 在 Keil µVision 打开双臂控制工程，确认目标器件为 `STM32F103C8`。
2. 给主控上电后，在 **Debug** 选 **ST-Link Debugger**，把 **Port** 设为 **SW**。读到目标芯片 ID 后继续。
3. 按 **F7** 编译，再按 **F8** 下载。看到 `Verify OK` 后，将 `BOOT0` 置于 `0`，按主板的 **RESET**。 [Keil 烧录说明](https://www.keil.com/support/man/docs/uv4cl/uv4cl_flash_programming.htm)

## 联动检查

先让示教臂的一个关节小幅动作，观察执行臂对应的关节。确认方向和行程后，再联动多个关节和夹爪。若舵机持续抖动或主控复位，优先检查舵机供电、共地以及关节是否卡住。
