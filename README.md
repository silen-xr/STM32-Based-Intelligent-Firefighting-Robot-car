# STM32-Based-Intelligent-Firefighting-Robot-car
## 综述
该智能灭火小车小车基于[STM32F103C8T6](https://item.szlcsc.com/datasheet/STM32F103C8T6/9243.html?c=Q4&msclkid=bd210d5676fb187a2b386654ae9f5593&jlc_vid=RFhdA1QCQwVfVVZeRQJeUFFRQQdZUgJSQAVfXlFRQgUxVlNfQlJbV1FQR1lWVjtWKAkaExQVTU5BDhADGk8dHQgFBAJABAsLSAwdBAgNHgVT)开发，主控板PCB与机械结构均为自行设计，代码以手搓为主，有AI的略微辅助。

未来更多开源分享，欢迎关注微信公众号：[流沙技术]

团队成员及分工：（在完成项目时补充）

# 智能灭火小车 · 硬件资料汇总
- 依据：`SCH_firefightingcar_2026-11-27.pdf`（单页 A4，立创EDA 专业版）
- 板子：100 × 100 mm 双层 1.6 mm，四角 M3（孔距边 8 mm），贴片全 0805 单面
- 电源链路：`DC_IN` → 3A 保险 → `DC_F` → 三脚开关(1→2) → `VM`
- 版本：v1.0（2026-11-27）
---
## 1. 元器件清单

| 类别 | 名称 / 规格 | 数量 | 位号 / 用途 | 备注 |
|---|---|---|---|---|
| 主控 | STM32F103C8T6 蓝板（LQFP48 / 64KB / 8MHz 晶振） | 1 | U1 | 用两排 1×20 插座插上 |
| 驱动 | TB6612FNG 双路电机驱动模块 | 1 | 2×8 排母 | 双路 1.2A 持续 |
| 电源 | MP1584 12V→5V 3A 降压模块 | 1 | U9 | 调到 5.0V 后锁死电位器 |
| 电源 | DC005 电源母座（5.5×2.1，内正外负） | 1 | DC1 | |
| 电源 | 自恢复保险 3A / 24V，BSMD1812-300 | 1 | U7 | 贴片，直接焊 |
| 电源 | 船型开关 KCD1-102N-R（3 脚带灯，15×21 面板式） | 1 | U10（建议改 SW1） | 面板外置，板上只留 3 焊盘 |
| 端子 | KF128-5.08-2P 直插 | 3（买 5 备） | U4 风机、U5 电机 A、U6 电机 B | |
| 电阻 | 0805 10k ±1% | 7 | R1 栅极下拉、R3 电池分压上、R5/R7/R9/R11/R13 循迹串联 | |
| 电阻 | 0805 20k ±1% | 5 | R6/R8/R10/R12/R14 循迹对地 | |
| 电阻 | 0805 3.3k ±1% | 1 | R4 电池分压对地 | |
| 电阻 | 0805 1k ±1% | 1 | R2 LED 限流 | |
| 电容 | 电解 100µF/25V，6.3×5.4 | 1 | C1（VM 滤波，贴 TB6612） | 见 §6 推荐料号 |
| 电容 | 电解 100µF/16V，6.3×5.4 | 2 | C5 舵机1、C6 舵机2 | |
| 电容 | 陶瓷 100nF/50V X7R 0805 | 1 | C2（与 C1 并联） | |
| 电容 | 陶瓷 10µF/25V X5R 0805 | 1 | C3（MP1584 输入） | |
| 电容 | 陶瓷 22µF/16V X5R 0805 | 1 | C4（MP1584 输出） | |
| 半导体 | MOSFET AO3400A（SOT-23） | 1 | Q1 风机低边开关 | 1=G、2=S、3=D |
| 半导体 | 肖特基 SS34（SMA） | 1 | D1 续流 | 阴极→VM、阳极→FAN_D |
| 连接 | 1×20 排母 2.54，行距 **15.24 mm** | 2 | 蓝板左/右排 | 不是 22.86 |
| 连接 | 1×8 排母 2.54 | 2 | TB6612 两排 | |
| 连接 | 排针 1×3 2.54 | 4 | H2/H3/H4/H5 火焰 | |
| 连接 | 排针 1×3 **2.0 mm** | 2 | H7 舵机1、H8 舵机2 | PZ2.0 间距 |
| 连接 | 排针 1×4 2.54 | 6 | H6 超声、H9 OLED、H11 按键、H12 串口、U2/U3 编码器 | |
| 连接 | 排针 1×7 2.54 | 2 | H1 循迹、H10 颜色 | |
| 连接 | 排针 1×4 2.54（模块端） | 1 | U9 MP1584 | 四角焊盘也可 |
| 连接 | 测试点 RH-5019 | 1 | TP1（接 NRST） | |
| 感知 | 三线火焰传感器（DO 输出） | 4 | `FLAME_L1/L2/R1/R2` | 供 3.3V |
| 感知 | HC-SR04 超声 | 1 | H6 | 供 5V，Echo 直连 PB11 |
| 感知 | 5 路循迹 TCRT5000 | 1 | H1 | **供 5V**，OUT 经 10k/20k 分压 |
| 感知 | TCS34725 颜色传感器（0x29） | 1 | H10（7 针） | VIN 建议接 5V |
| 人机 | 0.96" OLED I²C（0x3C） | 1 | H9 | GND/3V3/SCL/SDA |
| 人机 | 有源蜂鸣器模块（低电平触发） | 1 | U1 | 供 3.3V |
| 人机 | 按键模块（带 LED） | 1 | H11 | KEY/GND/LED/GND |
| 执行 | MG513 减速电机（12V，13 线霍尔） | 2 | 左/右轮 | 减速比待确认（P30/P45/P60） |
| 执行 | MG90S 舵机 | 2 | 舵机1 转盘 PA8、舵机2 出球 PA11 | 5V 独立 + 100µF |
| 执行 | 12V 涡轮风机 5015（0.3–0.5A） | 1 | 经 Q1 低边开关 | 加风道 |
| 电源 | 12V 2800mAh 电池组（带 DC 头） | 1 | 进 DC005 | 必须配保险与开关 |
| 紧固 | M3 铜柱 + 螺丝 + 螺母 | 4 组 | 四角孔（孔距边 8 mm） | |
| 机械 | 车架、轮子、万向轮、球仓、风机风道 | 1 套 | 3D 打印 / 亚克力 | 需匹配 6 mm 轴 |

---

## 2. CubeMX 配置

| 页面 | 配置项 | 设置值 | 说明 |
|---|---|---|---|
| RCC | HSE | Crystal/Ceramic Resonator，8MHz | 蓝板板载晶振 |
| Clock | PLL 倍频 / SYSCLK | ×9 → 72MHz | AHB 72、APB1 36、APB2 72 |
| Clock | ADC 预分频 | PCLK2/6 = 12MHz | ADC 上限 14MHz |
| SYS | Debug | **Serial Wire** | 释放 PA15/PB3/PB4，必须 |
| TIM1 | CH1(PA8)、CH4(PA11) | PWM，Prescaler 71、Period 19999、Pulse 1500 | 20ms 周期 = 50Hz |
| TIM2 | Encoder Mode | Prescaler 0、Period 65535，CH1=PA0 / CH2=PA1 | 左轮 |
| TIM3 | CH1(PA6)、CH2(PA7) | PWM，Prescaler 0、Period 3599 | 72MHz/3600 = 20kHz |
| TIM4 | Encoder Mode | Prescaler 0、Period 65535，CH1=PB6 / CH2=PB7 | 右轮 |
| USART1 | PA9 / PA10 | 115200，**8E1** | bootloader 同格式 |
| I2C1 | 重映射 | **SCL=PB8、SDA=PB9**，100kHz | 避开 TIM4 的 PB6/PB7 |
| ADC1 | IN4（PA4） | 12 位，采样 55.5 cycles | 电池电压 |
| GPIO 输出 | 推挽 | PB12/PB13(AIN1/2)、PB14/PB15(BIN1/2)、PA5(FAN)、PA15(BUZZ)、PC13(LED) | |
| GPIO 输入 | 上拉 | PB0/PB1/PB3/PB4/PB5(循迹)、PA2/PA3/PC14/PC15(火焰)、PB11(ECHO)、PA12(KEY) | 火焰/循迹为数字量输出 |
| NVIC | SysTick | 必开；USART1 RX 可选 | 编码器、PWM 不需要中断 |

---

## 3. 编码器参数

| 参数 | 数值 | 说明 |
|---|---|---|
| 编码器线数 | 13 线（每相每转 13 脉冲） | MG513 霍尔 |
| 倍频方式 | 4 倍频（双通道双边沿） | TIM2/TIM4 编码器模式自动 |
| 电机轴每转 | 13 × 4 = **52** 计数 | 减速前 |
| 减速比 | P30 = 30 / P45 = 45 / P60 = 60（**待确认**） | 看铭牌或数输出轴 |
| 输出轴每转 | 52 × 减速比 | P30→1560，P45→2340，P60→3120 |
| 轮径 | 65 mm（实测为准） | 周长 ≈ 204.2 mm |
| 分辨率 | P30 → 0.131 mm/计数（7.6 计数/mm）；P45 → 0.087 mm/计数；P60 → 0.065 mm/计数 | 204.2 / 每转计数 |
| 16 位溢出 | 1560 计数/转 → 约 42 转才溢出 | 每 10 ms 读差值累加，别直接算差 |
| 速度环 | 10 ms 采样 | 差值 → mm/s → PI |
| 标定方法 | 直行 2 m 记录左右计数 | 求实际 counts/mm，写死进参数 |

---

## 4. 接线表

| 模块 | 模块引脚 | 接到 | 供电 | 备注 |
|---|---|---|---|---|
| DC 电源入口 | DC005 → `DC_IN` → U7 保险 → `DC_F` → U10 开关(1→2) → `VM` | — | 12V | 开关灯脚 pin3 → GND |
| TB6612 | VM | `VM` | 12V | 与 C1 同点 |
| TB6612 | PWMA / AIN1 / AIN2 | PA6 / PB12 / PB13 | VCC 3.3V | AO1/AO2 → 左电机 |
| TB6612 | PWMB / BIN1 / BIN2 | PA7 / PB14 / PB15 | | BO2/BO1 → 右电机（BO2 在前） |
| TB6612 | STBY | 3.3V | — | 常使能，停机靠 PWM=0 |
| 左编码器 | A / B | PA0 / PA1 | 3.3V | 非 5V 容忍 |
| 右编码器 | A / B | PB6 / PB7 | 3.3V | 同上 |
| 5 路循迹 | GND / 5V / OUT1–5 | GND / 5V / `TRKR1–5` →(10k 串 + 20k 地)→ PB0/PB1/PB3/PB4/PB5 | **5V** | 分压后 3.33V |
| 火焰 左1（H3） | DO / GND / 3V3 | `FLAME_L1` → PA2 | 3.3V | |
| 火焰 左2（H2） | DO / GND / 3V3 | `FLAME_L2` → PA3 | 3.3V | |
| 火焰 右1（H5） | DO / GND / 3V3 | `FLAME_R1` → PC14 | 3.3V | |
| 火焰 右2（H4） | DO / GND / 3V3 | `FLAME_R2` → PC15 | 3.3V | |
| HC-SR04 | 5V / TRIG / ECHO / GND | 5V / PB10 / PB11 / GND | 5V | Echo 靠 PB11 的 FT |
| OLED（H9） | GND / 3V3 / SCL / SDA | GND / 3V3 / `SCL1`(PB8) / `SDA1`(PB9) | 3.3V | 0x3C |
| TCS34725（H10） | VIN / GND / SCL / SDA | 5V / GND / `SCL1` / `SDA1` | 5V | 0x29；3V3、INT、LED 悬空 |
| 舵机1（H7） | GND / 5V / SIG | GND / 5V / `SERVO1` → PA8 | 5V 独立 + C5 | 按针号插，别按线色 |
| 舵机2（H8） | GND / 5V / SIG | GND / 5V / `SERVO2` → PA11 | 5V 独立 + C6 | 同上 |
| 蜂鸣器（U1） | 3V3 / GND / I/O | 3.3V / GND / `BUZZ` → PA15 | 3.3V | 低电平响 |
| 按键（H11） | KEY / GND / LED / GND | `KEY`→PA12 / GND / `LED`→PC13（经 1k） / GND | — | 内部上拉 |
| 风机（U4） | 1 / 2 | `VM`（12V）/ `FAN_D`（Q1 漏极） | 12V | 源极接 GND，D1 反并 |
| 电池分压 | `VM` →10k(R3)→ PA4 →3.3k(R4)→ GND | PA4 | 12V | 12.6V → 3.12V |
| 串口（H12） | 3V3 / RXD / TXD / GND | 3.3V / `UART_RX`(PA10) / `UART_TX`(PA9) / GND | 3.3V | 板端已交叉；烧录时 BOOT0=1 |
| MP1584（U9） | IN+ / IN− / OUT+ / OUT− | `VM` / GND / `5V` / GND | 12V→5V | C3 输入、C4 输出 |

---

## 5. 引脚分配

| 功能 | 引脚 | 外设 / 模式 | 方向 | 备注 |
|---|---|---|---|---|
| 左编码器 A / B | PA0 / PA1 | TIM2_CH1/CH2 编码器 | 输入 | 非 5V 容忍 |
| 右编码器 A / B | PB6 / PB7 | TIM4_CH1/CH2 编码器 | 输入 | 占 PB6/PB7 → I2C 必须重映射 |
| 电机 PWM A / B | PA6 / PA7 | TIM3_CH1/CH2，20kHz | 输出 | 72MHz/3600 |
| 电机方向 A | PB12 / PB13 | GPIO → AIN1/AIN2 | 输出 | |
| 电机方向 B | PB14 / PB15 | GPIO → BIN1/BIN2 | 输出 | |
| TB6612 STBY | — | 接 3.3V | — | 无空余脚 |
| 舵机1 转盘 | PA8 | TIM1_CH1，50Hz | 输出 | PSC 71 / ARR 19999 |
| 舵机2 出球 | PA11 | TIM1_CH4，50Hz | 输出 | USB_DM，禁用 USB |
| 循迹 1~5 | PB0 / PB1 / PB3 / PB4 / PB5 | GPIO 输入 | 输入 | PB3/PB4 需关 JTAG |
| 火焰 左1 / 左2 | PA2 / PA3 | GPIO 输入 | 输入 | `FLAME_L1` / `FLAME_L2` |
| 火焰 右1 / 右2 | PC14 / PC15 | GPIO 输入 | 输入 | `FLAME_R1` / `FLAME_R2` |
| 超声 Trig | PB10 | GPIO 输出 | 输出 | 3.3V 有效 |
| 超声 Echo | PB11 | GPIO 输入 | 输入 | FT，5V 直连 |
| I²C1 SCL | PB8 | I2C1 重映射 | 双向 | OLED + TCS34725 |
| I²C1 SDA | PB9 | I2C1 重映射 | 双向 | 0x3C / 0x29 |
| 电池检测 | PA4 | ADC1_IN4 | 输入 | 10k / 3.3k |
| 启动按键 | PA12 | GPIO 输入上拉 | 输入 | 另一端 GND |
| 蜂鸣器 | PA15 | GPIO 输出 | 输出 | 低电平触发，需关 JTAG |
| 风机 MOS | PA5 | GPIO 输出 | 输出 | 高电平开 |
| LED 绿 | PC13 | GPIO 输出 | 输出 | 串 1k，低电平亮 |
| 调试串口 | PA9 / PA10 | USART1 TX/RX | 双向 | 115200 8E1 |
| SWD | PA13 / PA14 | SWDIO / SWCLK | 双向 | 走蓝板 4P 头，底板只留 TP1 |
| 未使用 | PB2 | BOOT1 | — | 悬空 |
| 未使用 | VB / 左排 pin1 | — | — | 原理图已标 NC |
