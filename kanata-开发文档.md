# Kanata Vim 风格全局导航键盘系统

## Coding Agent 开发规格 / Development Specification

> **目标：交给 Codex、DeepSeek Harness、OpenCode 等 Coding Agent 执行开发。**
>
> 本文不是 Kanata 教程，而是一份面向 AI Coding Agent 的工程规格。Agent 应根据本文目标，自主查阅 Kanata 1.12.0 官方文档、编写配置、验证、测试并迭代修复。

***

# 1. 项目目标

使用 **Kanata 1.12.0** 实现一个全局 Vim 风格键盘导航层。

核心设计非常简单：

- 正常情况下，键盘行为完全保持原样。
- **CapsLock** 是唯一特殊的物理键。
- CapsLock：
  - **短按 → Esc**
  - **按住 → 激活 Nav Layer**
- Nav Layer 下，通过 Vim 风格按键完成方向移动、跳转、删除、浏览器前进后退等操作。
- 松开 CapsLock 后立即恢复普通键盘行为。

最终目标：

> 在不改变普通打字习惯的情况下，让用户可以仅依靠左手 CapsLock + 右手键位完成大量鼠标/方向键导航操作。

***

# 2. 技术范围

## 2.1 必须使用

- Kanata 1.12.0
- Windows
- Kanata Windows `winIOv2` 后端

优先使用：

```text
winIOv2
```

除非实际验证发现功能无法实现，否则不要主动引入：

- Interception
- wintercept
- 第三方键盘驱动
- AutoHotkey
- 其他常驻键盘重映射软件

***

# 3. Agent 的开发职责

Coding Agent 不应该只是“生成一份看起来合理的 `.kbd` 文件”。

必须按照工程流程完成：

```text
阅读需求
 ↓
检查 Kanata 1.12.0 官方文档
 ↓
设计 defsrc / deflayer / alias
 ↓
编写 .kbd
 ↓
运行 Kanata --check
 ↓
修复语法/配置错误
 ↓
实际启动 Kanata
 ↓
逐项测试键位行为
 ↓
根据测试结果修改
 ↓
再次 --check
 ↓
最终交付配置
```

尤其注意：

**不能根据 Kanata 其他版本的记忆直接猜语法。**

如果某个 action、参数或语法存在不确定性，应优先查询：

<https://jtroo.github.io/config-1.12.0.html>

并以 **Kanata 1.12.0 官方文档**为准。

Kanata 配置采用 S-expression 语法，`defsrc` 定义被 Kanata 处理的物理输入，而 `deflayer` 按相同顺序定义对应行为。

***

# 4. 总体键盘架构

只设计两个 layer：

```text
base
  │
  │ CapsLock 按住
  ▼
nav
  │
  │ CapsLock 松开
  ▼
base
```

即：

```text
普通键盘
   │
   ├── CapsLock tap
   │       ↓
   │      Esc
   │
   └── CapsLock hold
           ↓
       Nav Layer
           │
           ├── HJKL → 方向键
           ├── Y/O  → Home/End
           ├── U/I  → PageUp/PageDown
           ├── C    → CapsLock
           ├── W/B  → Ctrl+Right/Left
           ├── X    → Delete
           ├── ;/'  → Alt+Left/Right
           ├── G    → Ctrl+Home
           ├── Shift+G → Ctrl+End
           └── ,/M  → 鼠标滚轮
```

***

# 5. Base Layer

Base Layer 的原则：

> **除了 CapsLock 之外，所有键都必须保持原始行为。**

例如：

```text
A → A
B → B
C → C
...
H → H
J → J
K → K
L → L
...
```

不能因为定义了 Nav Layer 就重新设计整个键盘。

***

# 6. CapsLock Tap/Hold

CapsLock 是整个系统的核心。

要求：

```text
CapsLock 短按
    ↓
Esc
```

以及：

```text
CapsLock 按住
    ↓
激活 nav layer
```

Kanata 1.12.0 提供 `tap-hold` 以及 `tap-hold-press`、`tap-hold-release` 等不同 tap/hold 行为。Agent 必须根据实际需求选择合适的实现，而不是机械套用示例。

初始参数建议：

```text
tap timeout  = 180 ms
hold timeout = 180 ms
```

但这不是永久固定值。

Agent 应将其设计成容易调整的参数，例如：

```text
180
```

后续可以根据实际打字体验测试：

```text
160
180
200
220
```

***

# 7. Nav Layer 键位规格

## P0：核心导航

这些功能优先级最高，必须首先实现并测试。

### 7.1 HJKL

| Nav 键 | 输出    |
| ----- | ----- |
| H     | Left  |
| J     | Down  |
| K     | Up    |
| L     | Right |

目标：

```text
Caps + H → ←
Caps + J → ↓
Caps + K → ↑
Caps + L → →
```

这组按键必须具备正常的连续按键能力。

例如：

```text
Caps + H H H H
```

应该可以连续向左移动。

***

# 8. Home / End

| Nav 键 | 输出   |
| ----- | ---- |
| Y     | Home |
| O     | End  |

例如：

```text
Caps + Y → Home
Caps + O → End
```

***

# 9. PageUp / PageDown

| Nav 键 | 输出       |
| ----- | -------- |
| U     | PageUp   |
| I     | PageDown |

例如：

```text
Caps + U → PageUp
Caps + I → PageDown
```

***

# 10. C → CapsLock

Nav Layer 中：

```text
C → CapsLock
```

目的：

> 保留原本 CapsLock 的切换功能。

因此：

```text
Caps + C
```

应该向操作系统发送：

```text
CapsLock
```

需要测试：

- CapsLock 状态是否正常切换
- 连续执行是否正常
- 不会导致 Nav Layer 状态异常

***

# 11. 单词级移动

## W → Ctrl + Right

```text
Caps + W
    ↓
Ctrl + Right
```

用于：

> 向下一个单词移动。

***

## B → Ctrl + Left

```text
Caps + B
    ↓
Ctrl + Left
```

用于：

> 向上一个单词移动。

***

# 12. Delete

```text
Caps + X
    ↓
Delete
```

注意：

这里明确要求：

```text
Delete
```

不是：

```text
Backspace
```

***

# 13. 浏览器 / 文件管理器前进后退

## `;` → Alt + Left

```text
Caps + ;
    ↓
Alt + Left
```

主要用于：

- 浏览器后退
- Windows Explorer 后退

***

## `'` → Alt + Right

```text
Caps + '
    ↓
Alt + Right
```

主要用于：

- 浏览器前进
- Windows Explorer 前进

Agent 不需要实现浏览器逻辑。

这里只负责向操作系统/应用发送对应键组合。

***

# 14. Ctrl + Home / Ctrl + End

## G

```text
Caps + G
    ↓
Ctrl + Home
```

用于：

> 快速跳到文档/页面开头。

***

## Shift + G

```text
Caps + Shift + G
    ↓
Ctrl + End
```

用于：

> 快速跳到文档/页面末尾。

这是一个重要要求：

**不能简单地把 G 固定映射成 Ctrl+Home、再单独猜测 Shift+G。**

需要正确处理：

```text
G
Shift + G
```

两种输入状态。

Agent 应查阅 Kanata 1.12.0 中适合实现条件键状态/组合输出的机制，并选择实际可验证的实现。

***

# 15. 鼠标滚轮

## `,` → 鼠标滚轮向上

```text
Caps + ,
    ↓
Mouse Wheel Up
```

## `M` → 鼠标滚轮向下

```text
Caps + M
    ↓
Mouse Wheel Down
```

***

# 16. 惯性滚动

目标不是简单的：

```text
Caps + ,
→ 一个固定滚轮事件
```

而是希望实现类似触控板/浏览器惯性滚动的感觉：

```text
按住 Caps + ,
       ↓
开始滚动
       ↓
速度逐渐增加
       ↓
达到最大速度
```

以及：

```text
松开
 ↓
速度逐渐衰减
```

如果 Kanata 1.12.0 当前版本能够通过官方支持的 mouse wheel acceleration action 实现，应优先使用 Kanata 原生能力。

初始参数可以作为实验值：

```text
initial velocity ≈ 3
max velocity     ≈ 1200
acceleration     ≈ 1.15
deceleration     ≈ 0.93
```

**这些参数不是硬性规格。**

Agent 应实际测试，并允许后续调整。

如果某种实现只是“重复发送滚轮事件”，而不是实际意义上的加速/减速，应明确记录其行为，不要把它描述成真正的物理惯性滚动。

***

# 17. Nav Layer 中未定义的键

核心原则：

> **未定义的 Nav Layer 按键必须透明传递。**

也就是说：

```text
Caps + Q
Caps + E
Caps + R
Caps + T
Caps + P
...
```

不应该产生奇怪的副作用。

Kanata 的透明键机制可以用于让当前 layer 不改变对应输入。

特别注意：

**不要因为 Nav Layer 存在，就把整个键盘重新映射一遍。**

***

# 18. 完整功能表

最终目标：

| 按键             | 行为         |
| -------------- | ---------- |
| Caps tap       | Esc        |
| Caps hold      | Nav Layer  |
| Caps + H       | Left       |
| Caps + J       | Down       |
| Caps + K       | Up         |
| Caps + L       | Right      |
| Caps + Y       | Home       |
| Caps + O       | End        |
| Caps + U       | PageUp     |
| Caps + I       | PageDown   |
| Caps + C       | CapsLock   |
| Caps + W       | Ctrl+Right |
| Caps + B       | Ctrl+Left  |
| Caps + X       | Delete     |
| Caps + ;       | Alt+Left   |
| Caps + '       | Alt+Right  |
| Caps + G       | Ctrl+Home  |
| Caps + Shift+G | Ctrl+End   |
| Caps + ,       | Wheel Up   |
| Caps + M       | Wheel Down |

其余按键：

```text
transparent
```

***

# 19. 配置设计原则

Agent 应尽量让配置具有清晰的工程结构。

推荐结构：

```text
defcfg

defsrc

defvar

defalias

deflayer base

deflayer nav
```

如果某些结构确实没有必要，可以减少。

原则：

> 简单优先，不为了“炫技”使用复杂 Kanata 特性。

例如不要为了实现：

```text
Caps + H → Left
```

引入复杂 macro / sequence / fake key 体系。

***

# 20. Alias 使用原则

重复或者复杂 action 可以使用 `defalias`。

例如：

```text
@capnav
```

代表：

```text
Caps tap → Esc
Caps hold → nav
```

类似：

```text
@word-right
@word-left
@page-up
```

是否使用 alias 由 Agent 根据可读性决定。

原则：

> Alias 是为了提高可读性，而不是为了增加抽象层。

***

# 21. Windows 配置

目标运行方式：

```text
Kanata Windows + winIOv2
```

Agent 应根据实际使用的 Kanata Windows 可执行文件确定启动参数。

不要默认加入：

```text
--interception
```

或任何不必要的第三方驱动依赖。

***

# 22. 配置验证

每次修改 `.kbd` 后，必须优先执行 Kanata 的配置检查：

```text
kanata.exe --check --cfg <config.kbd>
```

Kanata 1.12.0 官方文档明确提供 `--check`，用于只检查配置而不正常运行。

开发流程必须遵循：

```text
修改
 ↓
--check
 ↓
无错误
 ↓
启动
 ↓
实际测试
```

不能：

```text
修改
 ↓
直接认为语法正确
```

***

# 23. Agent 测试矩阵

Agent 完成配置后，至少测试以下场景。

## 23.1 Base Layer

确认：

```text
A-Z
0-9
符号
Enter
Backspace
Tab
Shift
Ctrl
Alt
Win
```

行为没有被改变。

***

## 23.2 Caps Tap

测试：

```text
Caps
```

结果：

```text
Esc
```

连续快速：

```text
Caps Caps Caps
```

也应该表现合理。

***

## 23.3 Caps Hold

测试：

```text
Caps + H
Caps + J
Caps + K
Caps + L
```

确认方向键正确。

***

## 23.4 连续导航

测试：

```text
Caps + H H H H
Caps + J J J
Caps + L L L
```

确认不会只触发一次。

***

## 23.5 快速组合

测试：

```text
Caps + W
Caps + B
Caps + X
Caps + ;
Caps + '
```

确认不会出现 tap/hold 误判。

***

## 23.6 Shift + G

重点测试：

```text
Caps + G
Caps + Shift + G
```

确认：

```text
G       → Ctrl + Home
Shift G → Ctrl + End
```

而不是两个都输出同一个行为。

***

## 23.7 C

测试：

```text
Caps + C
```

确认 CapsLock 状态真的发生切换。

***

## 23.8 鼠标滚轮

测试：

```text
Caps + ,
Caps + M
```

检查：

- 滚动方向
- 滚动速度
- 连续按住
- 松开
- 再次使用

***

# 24. 测试应用

至少选择以下应用进行测试：

### 编辑器

例如：

```text
VS Code
```

测试：

- Home / End
- Ctrl + Left / Right
- Ctrl + Home / End
- Delete
- 方向键

### 普通文本编辑器

例如：

```text
Notepad
```

测试基础键盘行为。

### Terminal

例如：

```text
PowerShell
Windows Terminal
```

确认 Ctrl 组合不会出现异常。

### 浏览器

例如：

```text
Chrome
Edge
Firefox
```

测试：

```text
Alt + Left
Alt + Right
PageUp
PageDown
Home
End
```

### 文件管理器

测试：

```text
Alt + Left
Alt + Right
```

***

# 25. Emergency Exit

必须保留 Kanata 自带的紧急退出机制。

Kanata 官方文档说明，可以通过同时按住：

```text
Left Ctrl + Space + Esc
```

强制退出 Kanata，而且该机制发生在 Kanata 重映射之前，因此即使配置出现严重问题，也可以尝试使用该组合退出。

Agent 不应该覆盖或破坏这个机制。

***

# 26. 错误处理原则

如果 Agent 遇到类似：

```text
unknown action
unknown key
invalid configuration
unexpected token
wrong number of arguments
```

不得猜测解决。

应该：

1. 查看错误位置。
2. 查看 Kanata 1.12.0 官方文档。
3. 确认该 action 在 1.12.0 是否存在。
4. 确认参数数量和顺序。
5. 修改。
6. 再次运行 `--check`。

***

# 27. 不允许的实现方式

除非需求发生变化，否则不要加入：

- Home Row Mods
- Vimium
- Vimium C
- AutoHotkey
- PowerToys Keyboard Manager
- 全键盘重映射
- 应用程序自动化
- 窗口管理
- OCR
- GUI 自动点击
- 鼠标移动模拟
- Vim 编辑模式
- Vim command mode
- 自定义文本编辑器快捷键体系

本项目只负责：

> **Kanata + 全局 Nav Layer**

***

# 28. 不要过度设计

这是一个非常重要的开发约束。

不要因为 Kanata 支持：

- macro
- sequence
- fake key
- tap dance
- one-shot
- chords
- arbitrary code

就主动把它们全部加入。

当前项目只需要：

```text
Caps tap/hold
+
Nav Layer
+
少量组合键
+
鼠标滚轮
```

优先实现一个：

> 小、稳定、容易调试、容易修改的配置。

***

# 29. 后续扩展

第一版完成以后，再考虑：

```text
V2
├── 更好的滚轮体验
├── 调整 Caps tap/hold 时间
├── 更多文本导航
├── 更多编辑操作
└── 其他专用 layer
```

但这些都不属于第一阶段。

第一阶段唯一目标：

> **把当前 Nav Layer 稳定实现。**

***

# 30. Agent 最终交付物

Coding Agent 最终应该交付：

```text
kanata-nav.kbd
```

以及一份简短的：

```text
README.md
```

README 至少包含：

```text
1. 使用的 Kanata 版本
2. 如何启动
3. 如何检查配置
4. 如何修改 Caps tap/hold 时间
5. 完整键位表
6. Windows 注意事项
7. Emergency Exit
```

如果 Agent 修改了任何非显而易见的 Kanata 行为，应在 README 中解释原因。

***

# 31. 最终验收标准

项目只有在以下条件全部满足时才算完成：

### 功能

- [x] Caps tap = Esc
- [x] Caps hold = Nav Layer
- [x] HJKL 正常
- [x] Y/O 正常
- [x] U/I 正常
- [x] C = CapsLock
- [x] W/B 正常
- [x] X = Delete
- [x] ;/' 正常
- [x] G = Ctrl+Home
- [x] Shift+G = Ctrl+End
- [x] ,/M 滚轮正常
- [x] 未映射键透明

### 稳定性

- [ ] 普通键盘行为没有改变
- [ ] Caps tap/hold 没有明显误触
- [ ] 连续方向键正常
- [ ] 快速组合正常
- [ ] VS Code 正常
- [ ] Terminal 正常
- [ ] Browser 正常
- [ ] Explorer 正常

### 工程质量

- [ ] `--check` 通过
- [ ] 没有不必要的第三方依赖
- [ ] 配置结构清晰
- [ ] 关键参数容易调整
- [ ] 有 README
- [ ] 保留 Emergency Exit

***

# 32. 给 Coding Agent 的最终指令

你现在不是在回答“Kanata 应该怎么配置”。

你的任务是：

> **作为一个软件工程 Coding Agent，把本规格实现成一个经过验证的 Kanata 1.12.0 配置。**

执行顺序：

```text
1. 阅读本规格
2. 阅读 Kanata 1.12.0 官方配置文档
3. 检查当前工作目录中的 Kanata 文件和已有配置
4. 设计最简单可靠的实现
5. 创建/修改 .kbd
6. 使用 --check 验证
7. 修复所有配置错误
8. 启动 Kanata
9. 按照测试矩阵验证
10. 如果发现行为问题，继续迭代
11. 最终输出：
    - 配置文件
    - README
    - 实际测试结果
    - 尚未解决的问题（如果有）
```

**不要自行扩展需求。**

特别是：

> 不要加入 Home Row Mods、Vimium、AHK 或其他键盘/浏览器工具。

当前项目就是一个：

**Kanata + CapsLock Tap/Hold + Vim 风格 Nav Layer**

的独立工程。
