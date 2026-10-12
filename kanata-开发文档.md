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
运行 Kanata --check (.\kanata_windows_gui_winIOv2_cmd_allowed_x64.exe --check)
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
.\kanata_windows_gui_winIOv2_cmd_allowed_x64.exe --check --cfg <config.kbd>
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

# 29. 版本演进路线图

当前项目处于 V1，后续按以下优先级演进。每个版本独立交付，完成并稳定后再进入下一个版本。

```text
V1 (当前)
  └─ CapsLock tap/hold + Nav Layer
      │
      ├─ V2: NavLock —— 导航层锁定 (最优先，体验提升最明显)
      │
      └─ V3: Symbols Layer —— 编程符号层
```

## 29.1 V1 — 当前目标

**唯一目标：把当前 Nav Layer 稳定实现。**

具体内容见本规格所有前文章节。验收标准见第 31 节。

## 29.2 V2 — NavLock (导航层锁定) ✅ 已实现

**实现日期：** 2026-10-13

### 问题

当前 Nav Layer 必须 **全程按住 CapsLock** 才能保持激活。短时间导航还行，但连续做"移动 → 滚动 → 再移动"的组合动作时，小拇指长时间按压会累。

### 方案

增加一个 **NavLock** 机制：在 Nav Layer 中，按 `R` 键进入"持久导航模式"，松开 CapsLock 后 Nav Layer 仍然保持激活，再按 `R` 退出。

### 触发/解锁方式

- **进入 NavLock：** 按住 CapsLock（激活 Nav Layer）→ 按 **R** → 松开 CapsLock
- **退出 NavLock：** 在 NavLock 状态下按 **R**

### 实现方案

在 kanata.kbd 中新增一个 `navlock` 层，使用 `layer-switch` 动作在 `base` ↔ `navlock` 之间切换：

```text
base ──(CapsLock hold)──→ nav ──(按 R: layer-switch navlock)──→ navlock
                                                                    │
                                                                    └──(按 R: layer-switch base)──→ base
```

关键设计：`navlock` 层是 `nav` 层的复制版，但有两处差异：

| 位置 | nav 层 | navlock 层 | 原因 |
|:--|:--|:--|:--|
| CapsLock (第三行首) | `_` (透明) | `@capnav` (恢复 tap-hold) | NavLock 状态下 CapsLock 仍能当 Esc / hold nav 用 |
| R 键 (第二行第五个) | `@navlock-in` (layer-switch navlock) | `@navlock-out` (layer-switch base) | 同一个物理键 toggle 进出 |

### 为什么不用 layer-toggle

Kanata 源码确认 `layer-toggle` **就是** `layer-while-held` 的别名（两者指向同一个 Action::Layer），松开即失效。而 `layer-switch` 才是永久切换 base layer 的动作。所以：

| 动作 | 行为 | 适合场景 |
|:--|:--|:--|
| `layer-while-held` / `layer-toggle` | 按住激活，松开关闭 | CapsLock hold → Nav Layer (V1 现有) |
| `layer-switch` | 永久切换 base layer | NavLock 的进入/退出 |

### 新增 alias

```scheme
;; V2 NavLock
navlock-in  (layer-switch navlock)   ;; 在 nav 层按 R：切换到 navlock
navlock-out (layer-switch base)       ;; 在 navlock 层按 R：切回 base
```

### 验证的 Kanata 特性

- `layer-switch` 永久切换 base layer ✅
- `layer-while-held` (别名 `layer-toggle`) 按住激活松开关闭 ✅
- 多层叠加：`navlock` 层激活时，CapsLock 的 `layer-toggle nav` 仍然可以临时叠加 ✅
- transparent key (`_`) 在每层正确向上传递 ✅

### 验收标准（全部通过）

- [x] NavLock 激活后，松开 CapsLock，Nav Layer 仍然生效
- [x] NavLock 激活后，CapsLock tap 仍然输出 Esc
- [x] NavLock 按 R 退出，恢复 Base Layer 正常打字
- [x] NavLock 状态不会与 Nav Layer 激活状态混乱
- [x] 所有 V1 功能在 NavLock 模式下仍然正常工作
- [x] `kanata --check` 通过

### R 键选择理由

- `R` 读作 Lock 的首字母谐音，好记
- QWERTY 布局中 R 在左手食指位置，按住 CapsLock（小拇指）时左手食指够得到
- `navlock` 层里 R 的意义从 "按字母 R" 变成 "退出 NavLock"，语义清晰

## 29.3 V3 — Symbols Layer (编程符号层) ✅ 已实现

**实现日期：** 2026-10-13

### 问题

编写代码时，高频符号 `( ) [ ] { } / \` 都集中在键盘顶排或需要 Shift，每次按都要抬手或按两个键。

### 触发方式

- **按住右 Alt** → 激活 Symbols Layer（`layer-toggle symbols`）
- **松开右 Alt** → Symbols Layer 自动退出
- **短按右 Alt** → 保持原 AltGr 行为

### 最终键位设计 (左手)

**设计逻辑：右手按 Alt 做修饰，左手腾出来按符号 —— 和 Shift/修饰键双手配合一致（右手 Shift + 左手大写字母 / 左手 Ctrl + 右手快捷键）**

```
右 Alt 按住时的左手映射:

                e=(   r=)   t=/

           d=[    f=]    g=\

                c={   v=}
```

**手指模式（好记的关键）：**
- **中指列** (e, d, c) → **左括号** `( [ {` ← 一列都是开括号
- **食指列** (r, f, v) → **右括号** `) ] }` ← 一列都是闭括号
- **食指扩展** (t, g) → **斜杠** `/ \`

**最终只保留 9 个符号** —— `=` 去掉，因为顶排默认 `=` 打法更习惯。

### 实现方案

```scheme
;; 右 Alt tap/hold: tap→ralt, hold→Symbols Layer
symalt (tap-hold $tap-timeout $hold-timeout ralt (layer-toggle symbols))
```

base 层和 navlock 层的 ralt 位置改为 `@symalt`。nav 层 ralt 透明（Nav 临时激活时可以用右手切换符号层）。

symbols 层左手全部重新映射，右手全部透明（保留打字能力）。

### 重要发现：Kanata S-expression 中的特殊字符

**Kanata 配置是 S-expression（Lisp 语法）**，以下字符不能直接写在 deflayer 里：

| 不能直接写 | 正确 chord 格式 | 含义 |
|:--|:--|:--|
| `(` | `S-9` | Shift+9 |
| `)` | `S-0` | Shift+0 |
| `{` | `S-[` | Shift+[ |
| `}` | `S-]` | Shift+] |
| `+` | `S-=` | Shift+= |
| `*` | `S-8` | Shift+8 |
| `~` | `S-grv` | Shift+反引号 |
| `!` | `S-1` | Shift+1 |
| `:` | `S-;` | Shift+; |
| `?` | `S-/` | Shift+/ |
| `|` | `S-\` | Shift+\ |
| `#` | `S-3` | Shift+3 |

以下字符是合法 key name，可以直接写：`[ ] - = \ , . ;`

参考 Kanata issue #459：https://github.com/jtroo/kanata/issues/459

### 为什么用左手而不是右手

初始设计想把符号放在右手（因为符号高频），后来发现：

1. **双手配合更自然**：右手按 Alt 做修饰，左手按符号 —— 和 Shift/Ctrl 的使用模式完全一致
2. **右手不闲着**：右 Alt 按住时右手可以同时打字（例如 `右Alt+e → 左a → 右Alt+r` 输出 `(a)`）
3. 如果你习惯单手忙，右 Alt 按住右手按符号也不是不行，但修饰键双手配合是更底层的人体工程学直觉

### 验收标准（全部通过）

- [x] 按住右 Alt 时，符号层生效
- [x] 短按右 Alt 时，输出原始 AltGr 行为
- [x] 9 个符号全部正确输出
- [x] 左手打字母 + 右手按 Alt 的组合流畅
- [x] V1 Nav Layer 功能不受影响
- [x] V2 NavLock 功能不受影响
- [x] `kanata --check` 通过

### 本次迭代踩过的坑

1. **Chord 格式问题**：一开始直接写 `( ) { }` 报 Kanata 解析错误，必须用 S-xxx chord 格式
2. **乱码行**：手动编辑时曾在 alias 区末尾遗留了 `)|||||#{}+-` 脏数据，导致后面 deflayer 结构异常。最终靠严格的 `--check` + 逐行读文件发现
3. **对齐问题**：deflayer 每行的位置必须严格对应 defsrc，用 `_` 透明键占位不能省。建议每行注释 anchor（`;; row1: tab q w e r t ...`）方便自查

## 29.4 版本隔离原则

- **每个版本独立发布**，完成一个再开始下一个
- **不跳过版本**，不并行开发 V2+V3
- **每个版本都要重新跑一遍第 31 节的验收矩阵**，确保新增功能没有破坏已有功能
- 每个版本的实现细节在开始该版本时，补充本规格文档对应的章节

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
