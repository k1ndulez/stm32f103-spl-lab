# software_F1D — STM32F103 标准库练手工程

电赛（控制类）软件线的练习工程，用来打通「串口 → 上位机波形」这条链路。
基于 **ST 标准外设库（StdPeriph）**，不是 HAL / CubeMX 工程。

- 芯片：**STM32F103C8T6**（蓝板，64 KB Flash / 20 KB RAM）
- 主频：72 MHz（HSE 8 MHz × 9）
- 工具链：**Keil MDK**（工程文件 `MDK-ARM/std_test.uvprojx`），ST-Link 下载
- 代码风格：clang-format（配置见 `.clang-format`）

## 当前功能

主循环每 10 ms 通过 USART1 发一行两列 CSV：

```
正弦值,锯齿值\r\n        例：512,128
```

- 正弦：64 点查表，范围 0~1023
- 锯齿：每 10 帧 +4，到 1023 归零

上位机用 **SerialPlot** 接两路，应看到一条正弦 + 一条锯齿。

## 目录

```
Library/        ST 标准外设库（官方，勿改）
start/          启动文件、CMSIS、system_stm32f10x.c
user/           main.c、oled.c/h、font.c/h、中断向量表
MDK-ARM/        Keil 工程 + 自己写的模块（uart.c/h、Delay.c/h）
```

## 构建

Keil uVision 打开 `MDK-ARM/std_test.uvprojx`：

- `F7` 编译（应 0 Error / 0 Warning）
- `F8` 下载（ST-Link）

仓库里不包含编译产物（`Objects/`、`Listings/`、`*.axf`、`*.hex` 等已在 `.gitignore` 排除），
clone 后需要自己重新编译一次。

## 进度

| 阶段 | 内容 | 状态 |
| --- | --- | --- |
| W1 | USART1 + SerialPlot 两条曲线跑通 | ✅ |
| W2 | 串口抽成 `uart.c/h` 模块，`main` 不再直接操作寄存器 | ✅ |
| W2 | 发送改为环形缓冲（64 B）+ TXE 中断，不再阻塞主循环 | ✅ |

`uart1_send_string()` 是阻塞版，保留作对照；`uart1_send_string_better()` 是中断 + 环形缓冲版，
主循环调用它只做「拷进 buffer 就返回」。

## 踩过的坑

- **SerialPlot 的 `Data Format → Column Delimiter` 必须点内置的 `comma`**，
  用 `other: ,` 会出现 Text View 空白、只画一个通道。
- 文件末尾必须有换行，否则 Keil 报 `#1-D: last line of file ends without a newline`。
- 只格式化自己的文件；`Library/`、`startup_stm32f10x_md.s`、`system_stm32f10x.c` 不要动。
