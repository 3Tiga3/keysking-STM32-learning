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

### 第 3 份 · 按键控制小灯

- **目标**：两组独立控制——PB12 按住时 PB14 亮、松开灭；PB13 每按一次 PB15 亮灭翻转（按一下亮、再按一下灭）
- **实现方式**：`HAL_GPIO_ReadPin` 读按键电平 + `HAL_GPIO_WritePin` / `HAL_GPIO_TogglePin` 控灯；翻转按键加 10ms 延时消抖与松手检测
- **涉及文件**：`Core/Src/main.c`、`Core/Src/gpio.c`、`keysking-STM32-learning.ioc`
- **要点**：PB12/PB13 配为上拉输入（松开为高电平、按下接地为低电平）；PB14/PB15 为高电平点亮；消抖用 10ms 延时二次确认，松手检测用空 `while` 防止一次按压触发多次翻转
- **Tag**：`hw03-key-led`

### 第 4 份 · 非阻塞延时流水灯

- **目标**：PB12、PB13、PB14 轮流点亮（500ms 间隔），但不用 `HAL_Delay` 阻塞
- **实现方式**：`HAL_GetTick()` 读取 SysTick 毫秒计数，主循环比对 `当前 tick - 上次 tick >= interval` 才切换灯，切换后更新 `last_tick`
- **涉及文件**：`Core/Src/main.c`、`Core/Src/gpio.c`、`keysking-STM32-learning.ioc`
- **要点**：SysTick 在后台自动计时，主循环不空转——CPU 空闲时可响应其他任务，是裸机多任务调度的核心思想；对比写法用减法而非相等判断，天然规避 tick 溢出问题；这一版 PB12/13/14 恢复为推挽输出
- **Tag**：`hw04-nonblock-flowing-led`

### 第 5 份 · 认识中断（EXTI 外部中断）

- **目标**：PB12 检测到下降沿触发外部中断，在中断里软件消抖后翻转 PB15 灯；主循环仍跑 PB14 的 2s 闪烁，两者并行
- **实现方式**：PB12 配 `GPIO_MODE_IT_FALLING`（下降沿触发），使能 `EXTI15_10_IRQn`；中断服务函数 `EXTI15_10_IRQHandler` 里先 `HAL_Delay(10)` 消抖，二次读引脚确认为低电平后才 `HAL_GPIO_TogglePin(PB15)`
- **涉及文件**：`Core/Src/gpio.c`、`Core/Src/stm32f1xx_it.c`、`Core/Src/main.c`、`keysking-STM32-learning.ioc`
- **要点**：
  - 中断是硬件自动跳转，CPU 不必轮询引脚——按下才响应，平时主循环全速跑自己的任务
  - 软件消抖：进中断后延时 10ms 再读一次，确认按键确实按下（低电平）才动作，滤掉机械抖动造成的误触发
  - **SysTick 优先级高于 EXTI15_10**：中断里调用 `HAL_Delay` 依赖 SysTick 继续计时，正因为 SysTick 优先级更高（能抢占本中断），中断里的 10ms 延时才能正常走完——这是能在 ISR 里用延时的前提
  - NVIC 里给 EXTI15_10 设了抢占优先级 15（最低），不影响系统节拍
- **Tag**：`hw05-exti`

## 学习进度

| 序号 | 主题 | 涉及文件 | 状态 |
| --- | --- | --- | --- |
| 1 | GPIO 输出 · 板载 LED 闪烁 | `Core/Src/main.c` | 已完成 |
| 2 | GPIO 输出 · 流水灯 | `Core/Src/main.c` | 已完成 |
| 3 | GPIO 输入 · 按键控灯 | `Core/Src/main.c`、`gpio.c` | 已完成 |
| 4 | 非阻塞延时 · SysTick 流水灯 | `Core/Src/main.c` | 已完成 |
| 5 | 外部中断 EXTI · 下降沿触发 | `Core/Src/stm32f1xx_it.c`、`gpio.c` | 已完成 |
| 6 | 定时器 / PWM | — | 待学 |
| 7 | USART 串口 | — | 待学 |
| 8 | DMA | — | 待学 |
| 9 | ADC | — | 待学 |
| 10 | I2C / SPI | — | 待学 |

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
