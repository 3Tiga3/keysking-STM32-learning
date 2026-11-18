# STM32F103C8T6 学习记录

跟着 keysking 的 STM32 教程做实验。代码持续更新，每次提交都是一个存档点，历史版本全部保留。

## 环境

- 芯片：STM32F103C8T6（LQFP48，蓝板 / 最小系统板）
- 工具链：CLion 2026.2.2 + STM32CubeMX（HAL 库，GCC arm-none-eabi 10.3.1）
- 板载 LED：PC13（绿色，低电平点亮）
- 工程配置：改 `keysking-STM32-learning.ioc` 后重新生成代码

## 作业记录

### 第 1 份 · 点灯大师

- **目标**：PC13 板载绿色 LED，间隔 0.5s（500ms）闪烁
- **实现方式**：`HAL_GPIO_WritePin` 翻转电平 + `HAL_Delay` 阻塞延时
- **涉及文件**：`Core/Src/main.c`、`Core/Src/gpio.c`、`Core/Inc/gpio.h`
- **要点**：PC13 低电平点亮、高电平熄灭；`HAL_Delay` 属阻塞式，期间 CPU 空转，不能处理其他任务
- **Tag**：`hw01-blink`

### 第 2 份 · 电灯宗师（流水灯）

- **目标**：PB12、PB13、PB14 三盏灯轮流点亮，形成流水效果
- **实现方式**：`state` 状态机（0~5 循环），每 300ms 切换一次，`HAL_GPIO_WritePin` 逐位置位/复位
- **涉及文件**：`Core/Src/main.c`
- **要点**：用状态机把"哪几盏亮"映射到 6 个状态，比逐个延时更清晰；`HAL_Delay` 仍是阻塞式
- **Tag**：`hw02-flowing-led`

## 学习进度

| 序号 | 主题 | 涉及文件 | 状态 |
| --- | --- | --- | --- |
| 1 | GPIO 输出 · 板载 LED 闪烁 | `Core/Src/main.c` | 已完成 |
| 2 | GPIO 输出 · 流水灯 | `Core/Src/main.c` | 已完成 |
| 3 | GPIO 输入 · 按键 | — | 待学 |
| 4 | 外部中断 EXTI | — | 待学 |
| 5 | 定时器 / PWM | — | 待学 |
| 6 | USART 串口 | — | 待学 |
| 7 | DMA | — | 待学 |
| 8 | ADC | — | 待学 |
| 9 | I2C / SPI | — | 待学 |

## 日常更新工作流

学完一节，回到本目录执行：

```bash
git add .
git commit -m "第N份-知识点"
git push
```

## 查看与找回历史

```bash
git log --oneline                          # 看全部存档点
git show <hash>                            # 看某次改了什么
git checkout <hash> -- Core/Src/main.c     # 把某文件恢复成历史版本
git revert <hash>                          # 撤销某次提交（已 push 的用这个）
```

## 说明

- `cmake-build-debug/`、`.idea/`、`*.bak` 及编译产物已在 `.gitignore` 中排除，本地文件不受影响。
- 撤销已推送的代码用 `git revert`，不要用 `git reset --hard`（会丢历史）。
