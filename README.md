# STM32-Based-Intelligent-Firefighting-Robot-car
## 综述
该智能灭火小车小车基于[STM32F103C8T6](https://item.szlcsc.com/datasheet/STM32F103C8T6/9243.html?c=Q4&msclkid=bd210d5676fb187a2b386654ae9f5593&jlc_vid=RFhdA1QCQwVfVVZeRQJeUFFRQQdZUgJSQAVfXlFRQgUxVlNfQlJbV1FQR1lWVjtWKAkaExQVTU5BDhADGk8dHQgFBAJABAsLSAwdBAgNHgVT)开发，主控板PCB与机械结构均为自行设计，代码以手搓为主，有AI的略微辅助。

未来更多开源分享，欢迎关注微信公众号：[流沙技术]

团队成员及分工：（在完成项目时补充）

## 小车制作所用元器件
| 序号 | 器材 | 规格 / 型号 | 数量 | 对应任务 | 备注 |
| --- | --- | --- | --- | --- | --- |
| 1 | STM32F103C8T6 开发板 | 蓝板 LQFP48 / 64KB Flash | 1 | 全车控制 |  |
| 2 | USB 转 TTL CH340 | CH340 | 1 | 串口烧录 / 监视 | UART 烧录需 BOOT0=1，且全片擦除 |
| 3 | TB6612FNG 驱动模块 | 双路 1.2A 持续 | 1 | 双电机驱动 | MG513 堵转电流可能超限，需软启动 |
| 4 | MG513 减速电机 | 12V / 13 线霍尔编码器 / 6mm 轴 | 2 | 左右驱动轮 | 减速比待确认（P30/P45/P60） |
| 5 | 12V 2800mAh 电池组 | 含 DC 公母头 + 充电器 | 1 | 主电源 | 必须加开关与 3A 保险丝 |
| 6 | DC-DC 可调降压模块 | 3A | 1 | 5V 稳压 | 确认输入范围可吃 12V |
| 7 | 多路电源模块 | 3A 大电流 / 多路输出 | 1 | 多路供电 | 规格待确认 |
| 8 | 5 路循迹模块 | TCRT5000 ×5，数字输出 | 1 | 引导线 + 火源弧线门控 | 供 3.3V；安装离地 8–15mm |
| 9 | 三线火焰传感器 | 只有 DO 数字量 | 2 | 角火检测 | 4 路规划：现有 2，另购 2 个同款 |
| 10 | HC-SR04 超声波 | 5V 供电 / Echo 5V | 1 | 测距避障 | Echo 接 PB11（5V 容忍） |
| 11 | 0.96 寸 OLED | 四脚 I2C，0x3C | 1 | 火源编码显示 | 与 TCS34725 共用 I2C1 |
| 12 | 有源蜂鸣器模块 | 低电平触发 | 1 | 火警 ≥1s | 接 PA15 |
| 13 | 按键模块 | 数字输出 | 1 | 启动 / 自检 | 接 PA12，内部上拉 |
| 14 | 3mm LED 红/黄/绿 | 直插 | 3 | 状态指示 | PC13–15 只能 3mA，串 1kΩ |
| 15 | 杜邦线 | 公对公/母对母 | 1 | 接线 | 电机大电流不要走细线 |
| 16 | 1/4W 金属膜电阻包 | 常用阻值 | 1 | 分压 / 限流 / 上拉 | 电池检测用 10k/3.3k |
| 17 | 100µF 25V 电解电容 | 直插 | 4 | 电源滤波 | 每个电机端子并一颗 |
| 18 | 车架 / 轮子 / 万向轮 | 3D 打印或亚克力 | 1 | 整车 | 需匹配 MG513 6mm 轴 |
| 19 | 3D 打印 / 亚克力加工 | 自有条件 | 1 | 结构件加工 | 用于球仓、风机风道、支架 |
| 20 | 12V 涡轮风机 | 5015 / 12V 0.3–0.5A | 1 | 风扇灭火 30 分 | 选带风道的型号 |
| 21 | MOSFET 驱动模块 | IRLZ44N 逻辑电平或继电器 | 1 | 驱动风机 | 并 1N4007 续流 |
| 22 | TCS34725 颜色传感器 | I2C 0x29，带白色补光 | 1 | 标定 B 档 25–50 分 | 装遮光暗室 |
| 23 | 舵机 | SG90 / MG90S ×2 | 2 | 转盘 + 出球门 | 5V 独立供电 |
| 24 | 电源开关 + 保险丝座 | 3A 保险丝 | 1 | 安全强制项 | 规则要求明显电源开关 |
| 25 | VL53L0X ToF | I2C ×2 | 2 | 沿墙 / 停距 | 可替代超声波做主测距 |
| 26 | 火焰传感器（3 线同款） | 3.3–5V DO | 4 | 扩到 4 路 | 与现有两个同款 |
| 27 | 白色 LED + 限流电阻 | 5mm 白光 | 1 | 颜色识别补光 | 现有电阻包可限流 |
| 28 | 1N4007 二极管 | 直插 | 5 | 风机续流 | 必备 |
| 29 | 自制主控板 PCB | 嘉立创打样 + SMT | 1 | +10 分 | 需交原理图/PCB |

## 引脚分配

| 功能 | STM32 引脚 | 外设 / 模式 | 方向 | 备注 |
| --- | --- | --- | --- | --- |
| 左编码器 A / B | PA0 / PA1 | TIM2_CH1/CH2 编码器模式 | 输入 | PA0/PA1 非 5V 容忍；编码器供 3.3V |
| 右编码器 A / B | PB6 / PB7 | TIM4_CH1/CH2 编码器模式 | 输入 | 与 I2C1 默认脚冲突，需重映射 I2C1 |
| 电机 PWM A / B | PA6 / PA7 | TIM3_CH1/CH2，20kHz | 输出 | 72MHz / 3600 |
| 电机方向 A | PB12 / PB13 | GPIO 输出 AIN1/AIN2 | 输出 |  |
| 电机方向 B | PB14 / PB15 | GPIO 输出 BIN1/BIN2 | 输出 |  |
| TB6612 STBY | — | 接 3.3V | — | 无空余脚，软件靠 PWM=0 停机 |
| 舵机 1（转盘） | PA8 | TIM1_CH1，50Hz | 输出 | 72MHz 预分频 71，ARR 19999 |
| 舵机 2（出球门） | PA11 | TIM1_CH4，50Hz | 输出 | PA11 为 USB_DM，不启用 USB |
| 5 路循迹 | PB0/PB1/PB3/PB4/PB5 | GPIO 输入 | 输入 | PB3/PB4 需关 JTAG 留 SWD |
| 火焰 1 / 2 | PA2 / PA3 | GPIO 输入（DO） | 输入 | 非 5V 容忍，模块供 3.3V |
| 火焰 3 / 4 | PC14 / PC15 | GPIO 输入（DO） | 输入 | 原黄/红 LED 脚，改作火焰 |
| HC-SR04 Trig | PB10 | GPIO 输出 | 输出 | 3.3V 触发有效 |
| HC-SR04 Echo | PB11 | GPIO 输入 | 输入 | PB11 为 FT，5V 回波可直接接 |
| OLED + TCS34725 | PB8 / PB9 | I2C1 重映射 SCL/SDA | 双向 | 0x3C / 0x29 |
| 电池电压检测 | PA4 | ADC1_IN4 | 输入 | 10k/3.3k 分压，12.6V→3.12V |
| 启动按键 | PA12 | GPIO 输入上拉 | 输入 | 另一端接 GND |
| 蜂鸣器 | PA15 | GPIO 输出 | 输出 | 低电平触发；需关 JTAG |
| 风机 MOSFET | PA5 | GPIO 输出 | 输出 | 12V 直供，加续流二极管 |
| LED 绿 | PC13 | GPIO 输出 | 输出 | 黄/红 LED 取消，脚位让给火焰 3/4 |
| 调试串口 | PA9 / PA10 | USART1 TX/RX | 双向 | 建议 115200 8E1，与 bootloader 一致 |
| SWD 下载 | PA13 / PA14 | SWDIO / SWCLK | 双向 | 保留，不要占用 |

## 接线表

| 模块 | 模块引脚 | 接到 | 电源 | 备注 |
| --- | --- | --- | --- | --- |
| TB6612FNG | PWMA / AIN1 / AIN2 | PA6 / PB12 / PB13 | VCC 3.3V | 左电机 AO1/AO2 |
| TB6612FNG | PWMB / BIN1 / BIN2 | PA7 / PB14 / PB15 | VM 12V | 右电机 BO1/BO2 |
| TB6612FNG | STBY | 3.3V | — | 常使能 |
| 左编码器 | A / B | PA0 / PA1 | 3.3V | 若必须 5V，A/B 各加 10k/20k 分压 |
| 右编码器 | A / B | PB6 / PB7 | 3.3V | 同上 |
| 5 路循迹 | OUT1–OUT5 | PB0/PB1/PB3/PB4/PB5 | 3.3V | 按实物从左到右对应 |
| 火焰 ×4 | DO | PA2/PA3/PC14/PC15 | 3.3V | 3.3V 灵敏度略低，火源区内够用 |
| HC-SR04 | Trig / Echo | PB10 / PB11 | 5V | Echo 5V 靠 PB11 的 FT |
| OLED | SCL / SDA | PB8 / PB9 | 3.3V | I2C1 重映射 |
| TCS34725 | SCL / SDA | PB8 / PB9 | 3.3V | 与 OLED 并接，地址 0x29 |
| 舵机 ×2 | 信号 | PA8 / PA11 | 5V 独立 | 不要从 3.3V 取电，加 100µF |
| 风机 | MOSFET 栅极 | PA5 | 12V | 漏极接风机负极，源极接 GND |
| 按键 | 一端 / 另一端 | PA12 / GND | — | 内部上拉 |
| 蜂鸣器 | I/O | PA15 | 3.3V | 低电平响 |
| LED 绿 | 阳极经 1kΩ | PC13 | 3.3V | 低电平点亮 |
| CH340 | TX / RX | PA10 / PA9 | 3.3V 档 | 烧录时 BOOT0=1 |
| 电池分压 | 12V → 10k → PA4 → 3.3k → GND | PA4 | 12V | 分压后 12.6V≈3.12V |

## 编码器参数

| 参数 | 数值 | 说明 |
| --- | --- | --- |
| 编码器线数 | 13 线（每相每转 13 个脉冲） | MG513 霍尔编码器规格 |
| 倍频方式 | 4 倍频（TI12，双通道双边沿） | TIM2 / TIM4 编码器模式 |
| 电机轴每转计数 | 13 × 4 = 52 | 减速前的电机轴 |
| 减速比 | 待确认：P30=30、P45=45、P60=60 | 看电机铭牌或数输出轴转数 |
| 输出轮每转计数 | 52 × 减速比 | P30→1560，P45→2340，P60→3120 |
| 轮径假设 | 65mm（以实物为准） | 周长 = π × 65 ≈ 204.2mm |
| 分辨率（P30） | 0.131 mm/计数 | 204.2 / 1560；可跑 7.6 计数/mm |
| 分辨率（P45） | 0.087 mm/计数 | 204.2 / 2340 |
| 分辨率（P60） | 0.065 mm/计数 | 204.2 / 3120 |
| 16 位溢出 | 1560 计数/转 → 约 42 转溢出 | 每 10ms 采样并累加，勿直接读 16 位差值 |
| 速度环 | 每 10ms 读一次编码器差值 | 换算成 mm/s 后做 PI |
| 标定方法 | 直行 2m，记录左右计数差 | 求出实际 counts/mm，写进参数表 |

## CubeMX配置

| 页面 / 外设 | 配置项 | 设置值 | 备注 |
| --- | --- | --- | --- |
| RCC | HSE | Crystal/Ceramic Resonator（8MHz） | 蓝板板载 8MHz 晶振 |
| Clock | PLL 倍频 / SYSCLK | ×9 / 72MHz | AHB 72、APB1 36、APB2 72 |
| Clock | ADC 预分频 | PCLK2/6 = 12MHz | ADC 不得超过 14MHz |
| SYS | Debug | Serial Wire | 释放 PA15/PB3/PB4 |
| TIM1 | 通道 / 频率 | CH1(PA8)、CH4(PA11)，50Hz | Prescaler 71，Period 19999，Pulse 1500 |
| TIM2 | 模式 | Combined Channels = Encoder Mode | Prescaler 0，Period 65535 |
| TIM3 | 通道 / 频率 | CH1(PA6)、CH2(PA7)，20kHz | Prescaler 0，Period 3599 |
| TIM4 | 模式 | Encoder Mode | Prescaler 0，Period 65535 |
| USART1 | 参数 | 115200 / 8E1 | 与插件串口 8E1 一致，bootloader 也用 PA9/PA10 |
| I2C1 | 引脚重映射 | SCL=PB8，SDA=PB9，100kHz | 避开 TIM4 的 PB6/PB7 |
| ADC1 | 通道 | IN4（PA4） | 12 位，采样时间 55.5 cycles |
| GPIO | 输出标签 | FAN、BUZZER、AIN1/2、BIN1/2、LED_G/Y/R | 见“引脚分配”表 |
| GPIO | 输入标签 | TRACK1–5、FLAME_L/R、ECHO、KEY | 上拉/浮空按模块输出类型 |
| NVIC | 中断 | SysTick 必须开；USART1 RX 中断可选 | 编码器与 PWM 不需要中断 |
