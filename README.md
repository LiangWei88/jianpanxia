# Jianpanxia · 键盘侠

> An intuitive way to minimize finger movements on **any** keyboard — no firmware flashing, no layout swapping, keep your typing habits intact.

只靠 **两个键**（CapsLock + 右 Alt），让你在 **不改变打字习惯** 的前提下，把导航和编程符号从"抬手去顶排"变成"主行就能按到"。

```
┌─────────────────────────────────────────────────────────────────┐
│  Base Layer · 正常打字                                         │
│  除 CapsLock 和右 Alt 外，所有键保持原始行为                   │
│                                                                 │
│  ┌── CapsLock ──┐              ┌── 右 Alt ──┐                  │
│  │ tap → Esc     │              │ tap → AltGr │                 │
│  │ hold ──────┐  │              │ hold ──────┐ │                │
│  └─────────────┼──┘              └─────────────┼─┘               │
│                │                                │                │
│  ┌─────────────▼──────────┐    ┌───────────────▼──────────┐    │
│  │  Nav Layer (Caps hold) │    │  Sym Layer (右 Alt hold)  │    │
│  │  HJKL 方向移动         │    │  左手 9 个编程符号        │    │
│  │  Y/O 行首尾            │    │  e=(  r=)  t=/           │    │
│  │  U/I 翻页              │    │  d=[  f=]  g=\           │    │
│  │  W/B 单词移动          │    │  c={  v=}               │    │
│  │  ;/' 浏览器后退前进    │    │                           │    │
│  │  R 进入 NavLock        │    └───────────────────────────┘    │
│  └────────────────────────┘                                     │
└─────────────────────────────────────────────────────────────────┘
```

> [!NOTE]
> **Kanata v1.12.0** · **Windows** · 无需刷固件，无需改键盘布局，开箱即用。

---

## 目录

- [核心理念](#核心理念)
- [三层总览](#三层总览)
  1. [Base Layer — 正常打字](#1-base-layer--正常打字)
  2. [Nav Layer — Vim 风格全局导航](#2-nav-layer--vim-风格全局导航)
  3. [NavLock — 导航层锁定 (v2)](#3-navlock--导航层锁定-v2)
  4. [Symbols Layer — 左手编程符号 (v3)](#4-symbols-layer--左手编程符号-v3)
- [3 分钟上手](#3-分钟上手)
- [可调参数](#可调参数)
- [FAQ](#faq)
- [和 Arsenik 有什么关系？](#和-arsenik-有什么关系)
- [紧急退出](#紧急退出)

---

## 核心理念

> 这个项目的起点，不是什么宏大的键盘工程计划，只是一个 Vim 用户的日常小抱怨。

### 故事从这里开始

**很多年前**（久到记不清哪篇文章了），我看到有人说：

> 键盘上最没用的键是 CapsLock。把它改成 Esc 怎么样？

作为一个 Vim 用户，这简直是刚需 —— 在 Vim 里按 Esc 退出编辑模式是每秒要做几十次的事，而 CapsLock 大概一年才用两次。

于是我用 **AutoHotkey v1** 写了最早的版本：

```
CapsLock 短按 → Esc
CapsLock 按住 → HJKL 变成方向键
```

**就这两行，用了好几年。**

后来慢慢加了东西 —— Home/End、PageUp/PageDown、单词级移动、浏览器后退前进、鼠标滚轮。再后来 **AHK v1 官方宣布停止维护**，于是迁移到了 [Kanata](https://github.com/jtroo/kanata)（Rust 写的，更稳、更快、跨平台）。

再然后我看到了 **[Arsenik](https://github.com/OneDeadKey/arsenik)**。它的 NavLock（导航层锁定）和符号层设计让我眼前一亮 —— 原来还能这么玩。于是 V2 加了 NavLock，V3 加了左手符号层。

### 所以 Jianpanxia 是什么？

**是一条 Vim 用户的键盘折腾时间线：**

```
Vim 导航习惯
  │
  └── CapsLock 改 Esc（老文章灵感）
        │
        └── + HJKL 方向（AHK v1 实现，V1）
              │
              └── + NavLock（借鉴 Arsenik，V2）
                    │
                    └── + 左手符号层（借鉴 Arsenik，V3）
                          │
                          └── 迁移到 Kanata（AHK v1 停维护）
```

### 解决什么问题？

你每天写代码、刷网页、逛终端时，有多少操作在让手指"往返"？

- 在编辑器里移动光标 → 右手离开主行按方向键
- 打 `( ) { } [ ]` → 抬手到顶排 + 按 Shift
- 浏览器后退 → 伸手去按 Alt+左箭头
- 翻页 → 去按 PageUp / PageDown

传统键盘布局是打字时代设计的，为纯文本输入优化。但现代编程和 Web 浏览，导航和符号操作占了近一半。

**Jianpanxia 的做法很简单：**

> 不改你的键盘，**只加两层"隐身模式"** —— 按住 CapsLock 进入 Nav 模式，按住右 Alt 进入 Sym 模式。松开立即恢复正常。

**三层优势：**

- **零学习成本** — 如果你用过 Vim/Vimium，HJKL、W/B 一看就懂
- **全应用覆盖** — VS Code、浏览器、终端、文件管理器、设置面板统一生效
- **不破坏习惯** — Base Layer 完全保留，除了 CapsLock → Esc 补偿，一切照旧
- **零硬件门槛** — 任何 Windows 键盘都能用，无需刷固件

---

## 三层总览

### 整体架构

```
Base Layer (正常打字)
  │
  ├── 短按 CapsLock  ──→ Esc
  │
  ├── 按住 CapsLock  ──→ Nav Layer (V1)
  │     │
  │     └── 按 R     ──→ NavLock (V2，持久激活)
  │
  └── 按住 右 Alt    ──→ Symbols Layer (V3)
```

### 一张图看懂

```
                  ┌─── 右 Alt 按住 ─── Symbols Layer ───┐
                  │                                      │
   ┌──────────────┤                                      ├──────────────┐
   │ Base Layer   │                                      │              │
   │              │                                      │              │
   │  Caps tap → Esc                                     │              │
   │  Caps hold ──→ Nav Layer ──→ R ──→ NavLock          │              │
   │              │                                      │              │
   └──────────────┴──────────────────────────────────────┴──────────────┘
```

---

### 1. Base Layer — 正常打字

**除了 CapsLock 和右 Alt 之外，所有键保持原始行为。**

```
 ` 1 2 3 4 5 6 7 8 9 0 - = ⌫
 ⇥ q w e r t y u i o p [ ] \
 ☒ a s d f g h j k l ; ' ⏎     ← CapsLock 在这里做特殊处理
 ⇧ z x c v b n m , . / ⇧
 ⌃ ⌥ ⌘   ␣   ⌥ ⌘ ⌃           ← 右 Alt 在这里做特殊处理
```

| 键 | tap (短按) | hold (按住) |
|:--|:--|:--|
| **CapsLock** | `Esc` | Nav Layer |
| **右 Alt** | 保持 AltGr 原行为 | Symbols Layer |

> CapsLock → Esc 是 Vim 用户和终端重度使用者的日常刚需，顺便补偿了 CapsLock 被"挪用"的遗憾。

---

### 2. Nav Layer — Vim 风格全局导航

**按住 CapsLock 激活，松开立即恢复。**

```
                    ⌫     ⌫
                   PgUp  PgDn  End
            ⌫
   Ctrl+Left  ←  ↓  ↑  →  Alt+Left  Alt+Right
              Del  Caps  Ctrl+Home
                    ↑
           滚轮↓   Ctrl+Left   滚轮↑
                 ↑  ↑
                 R: 按此进入 NavLock
```

#### 完整键位表

| Caps + 键 | 输出 | 说明 |
|:--|:--|:--|
| **H J K L** | ← ↓ ↑ → | Vim 风格方向移动 |
| **Y / O** | Home / End | 行首 / 行尾 |
| **U / I** | PageUp / PageDown | 翻页 |
| **W / B** | Ctrl+Right / Ctrl+Left | 下一个 / 上一个单词 |
| **G** | Ctrl+Home | 文档开头 |
| **Shift+G** | Ctrl+End | 文档末尾 |
| **X** | Delete | 向后删除 |
| **C** | CapsLock | 保留原切换功能 |
| **; / '** | Alt+Left / Alt+Right | 浏览器 / 文件管理器后退前进 |
| **, / M** | 滚轮↑ / 滚轮↓ | 带惯性加速 |
| **R** | → NavLock | 进入导航锁定 |

> Nav Layer 中未列出的按键 **透明传递** —— 保持 Base Layer 的原始行为。比如 Caps + Q 就是普通 Q，Caps + 数字就是普通数字。

---

### 3. NavLock — 导航层锁定 (v2)

**问题**：长时间按 CapsLock 做连续导航，小拇指会累。

**解决方案**：按住 CapsLock 进入 Nav → 按 R 进入 NavLock → 松开 CapsLock，导航层仍然保持激活。

```
正常操作:  CapsLock hold → H → J → K → L (一直按着累)
NavLock:   CapsLock hold → R → 松开 → H J K L (不用按了!) → 再按 R 退出
```

#### 使用方式

```
进入 NavLock:  CapsLock hold (Nav 激活) → 按 R → 松开 CapsLock
退出 NavLock:  NavLock 状态下 → 按 R (切回 Base)
```

NavLock 激活时，所有 Nav Layer 功能照常生效 —— 包括滚轮、方向键、单词移动等。CapsLock 也恢复了正常的 tap-hold（tap=Esc, hold=Nav），所以你也可以在 NavLock 状态下短暂按 CapsLock 做其他事。

---

### 4. Symbols Layer — 左手编程符号 (v3)

**问题**：`( ) [ ] { } / \` 都在顶排或要 Shift，打代码时来回抬手很累。

**解决方案**：按住右 Alt，左手输出 9 个高频编程符号。**右手可以继续打字，双手配合。**

#### 键位设计

```
右 Alt 按住时的左手映射:

                 (   )   /        ← 上行: e=( r=) t=/

            [   ]   \             ← 主行: d=[ f=] g=\

                 {   }            ← 下行: c={ v=}
```

#### 好记的手指模式

| 手指 | 键位 | 符号类型 |
|:--|:--|:--|
| **左手中指** | e, d, c | `(` `[` `{`  ← 一列都是**开括号** |
| **左手食指** | r, f, v | `)` `]` `}`  ← 一列都是**闭括号** |
| **食指扩展** | t, g | `/` `\`  ← **斜杠** |

#### 为什么用左手而不是右手？

因为 **右手按 Alt 做修饰 + 左手按符号** 才是正确的人体工程学：

- 就像 **右 Shift 按住 → 左手打大写字母** 一样自然
- 就像 **左手 Ctrl 按住 → 右手按 C 复制** 一样自然
- 右 Alt 按住时，右手还能继续打字 —— 比如 `(a)` 可以在同一组合里完成：右Alt+e → 左a → 右Alt+r

**对比**：把符号放在右手意味着"右手既要按 Alt 又要按符号"，单手忙不过来，体验差很多。

---

## 3 分钟上手

### 1. 准备 Kanata

下载 Kanata Windows 可执行文件（推荐 winIOv2 后端，无需驱动）：

> [Kanata 官方 Release](https://github.com/jtroo/kanata/releases) · 找 `kanata_windows_gui_winIOv2_cmd_allowed_x64.exe`

### 2. 下载配置

克隆本仓库，或只下载 `kanata.kbd`：

```powershell
git clone https://github.com/LiangWei88/jianpanxia.git
cd jianpanxia
```

### 3. 启动

```powershell
.\kanata_windows_gui_winIOv2_cmd_allowed_x64.exe --cfg kanata.kbd
```

建议把启动命令做成 Windows 任务计划，开机自动运行。

### 4. 验证配置语法

修改 `kanata.kbd` 后，**务必先检查再启动**：

```powershell
.\kanata_windows_gui_winIOv2_cmd_allowed_x64.exe --check --cfg kanata.kbd
```

`--check` 静默无输出 = 通过。有输出 = 有错误。

---

## 可调参数

所有参数集中在 `kanata.kbd` 的 `defvar` 区块：

```scheme
(defvar
  tap-timeout   180   ;; tap 判定超时 (毫秒)
  hold-timeout  180   ;; hold 激活超时 (毫秒)
)
```

### 调整 tap/hold 灵敏度

| 场景 | 推荐值 | 说明 |
|:--|:--|:--|
| 快速打字者 | 160 ms | 响应快，容错稍低 |
| 默认 | 180 ms | 平衡值 |
| 保守用户 | 200–220 ms | 容错高，响应稍慢 |

两值相等时体验最稳定。修改后执行 `--check` 再重启。

### 鼠标滚轮惯性

```scheme
;; initial-velocity  max-velocity  acceleration  deceleration
(mwheel-accel-up   3 1200 1.15 0.93)
(mwheel-accel-down 3 1200 1.15 0.93)
```

- **initial-velocity**：初始滚动速度
- **max-velocity**：最大滚动速度上限
- **acceleration**：按住时每帧速度乘以这个数（>1 = 加速）
- **deceleration**：松开后每帧速度乘以这个数（<1 = 衰减）

想滚得更 "快" → 调大 `max-velocity`；想加速更 "猛" → 调大 `acceleration`。

---

## FAQ

### Kanata 是什么？和 AutoHotkey 有什么区别？

[Kanata](https://github.com/jtroo/kanata) 是用 Rust 写的跨平台键盘重映射工具。相比 AHK：

- **性能更好** — Rust 原生编译，延迟更低
- **语法更结构化** — S-expression，配置可维护性高
- **跨平台** — Windows / Linux / macOS 统一配置格式
- **AHK v1 已停维护** — Kanata 是更长远的选择

本项目最早用 AHK v1 实现，后来迁移到 Kanata。

### 会不会和 Vimium / Vimium C 冲突？

**完全不冲突**。Vimium 只在浏览器里生效，Jianpanxia 是操作系统级的。它们可以叠加使用：

- 浏览器导航：Vimium（j/k 滚动、f 链接提示）
- 编辑器 + 终端 + 全局：Jianpanxia（HJKL 方向、W/B 单词、Y/O 行首尾）

### 游戏里会不会被检测为作弊？

Kanata winIOv2 后端基于 Windows 原生 API（LLHOOK + SendInput），不安装内核驱动。大多数游戏反作弊不会检测它。

如果遇到问题，可以切换到 **Interception 后端**（需额外安装驱动），或者在游戏里退出 Kanata。

### 为什么不直接用 Arsenik？

见下文 [和 Arsenik 有什么关系？](#和-arsenik-有什么关系)

### 可以在 Mac 或 Linux 上用吗？

**Mac / Linux 用户**：Kanata 支持这些平台，但本项目的 `kanata.kbd` 还没在非 Windows 上测试过。理论上只需要换个后端（macOS: `karabiner`/`hidutil`，Linux: `evdev`），配置格式本身是跨平台的。欢迎贡献适配版本。

### CapsLock 原本的大写功能没了？

**没丢**。在 Nav Layer 里按 `Caps + C` 就能切换 CapsLock 状态。大写输入不受影响 —— Shift 照常工作。

### 紧急退出怎么办？

同时按住 **物理键** `Left Ctrl + Space + Esc`（这三个键不受 Kanata 重映射影响），Kanata 会立即退出，键盘恢复正常。详见 [紧急退出](#紧急退出)。

---

## 和 Arsenik 有什么关系？

**先澄清一个容易误会的事：Jianpanxia 不是 "Arsenik 的低配版" 或 "Arsenik 的 Windows 实现"。**

它们的起点完全不同：

```
Jianpanxia:  Vim 用户 → CapsLock 改 Esc → HJKL 方向 → 慢慢加功能 → 看到 Arsenik 借了几个好点子
Arsenik:     键盘工程爱好者 → 从 0 设计一套完整的 33 键替代体系（Angle Mod + Thumb-Taps + HRM + 四层）
```

**Jianpanxia 的 V1（CapsLock tap/hold + Nav Layer）在看到 Arsenik 之前就已经用了好几年了。**

### 那 Arsenik 给了我什么？

[Arsenik](https://github.com/OneDeadKey/arsenik) 是我最欣赏的键盘增强项目之一。它提出了一套完整的 33 键键盘替代方案 —— Angle Mod + Thumb-Taps + Home Row Mods + 四层（Nav/Sym/Num/FunMedia）。看完之后我拍了大腿：原来还能这样！

从 Arsenik **借鉴了三样东西**：

| 借鉴 | 说明 | 实现 |
|:--|:--|:--|
| **NavLock** | 导航层锁定 —— 不用一直按着修饰键 | V2：Caps hold → R → 松开，Nav 仍然激活 |
| **符号层用左手** | 右手按 Alt 修饰 + 左手按符号，双手配合 | V3：右 Alt hold → 左手 9 个编程符号 |
| **渐进式引入** | 先让一个功能跑通、手适应了，再加下一个 | V1→V2→V3 每次只加一层 |

### 那为什么不直接用 Arsenik？

因为 Arsenik **太激进了**。它是一套完整的 33 键键盘替代方案 —— 也就是说它假设你愿意 **彻底重构整个键盘布局**：

| 维度 | Arsenik | Jianpanxia |
|:--|:--|:--|
| **定位** | 33 键键盘替代方案 | 键盘增强层 |
| **学习成本** | Home Row Mods + Thumb-Taps + 全新按键 | 只有 CapsLock + 右 Alt 两个 tap-hold 键 |
| **对打字习惯的改变** | **改了** —— 左 Alt/右 Alt/Space 都被劫持 | **不改** —— Base Layer 完全保留 |
| **复杂度** | 5+ layer，完整体系 | 极简，3 个 layer |
| **tap-hold 键数量** | 3 个（左Alt / Space / 右Alt） | 2 个（CapsLock / 右Alt） |
| **适用场景** | 愿意彻底重构按键的进阶用户 | 想"加点功能但不想改习惯"的普通开发者 |

### 简单说

Arsenik 想要的是 **"Bring keys to your fingers" —— 让手指完全不离开主行**。为了达到这个目标，它改了整个键盘。

Jianpanxia 认同这个理念，但**不想改键盘本身**。所以只加了两层"隐身模式"。

两条路都对，只是目标不同 —— Arsenik 是完整方案，Jianpanxia 是增量改进。

### 哪些 Arsenik 功能我明确没抄

| 没抄 | 原因 |
|:--|:--|
| **Home Row Mods** | HRM 学习成本高，tap/hold 误判普遍，会彻底改变打字习惯。我的原则是"Base Layer 完全保留" |
| **Angle Mod** | ISO/ortholinear 专用，ANSI 用不上 |
| **三拇指 tap-hold** | 一个 tap-hold 键能搞定的事，不要加三个。tap-hold 键越多，边界 case 越多 |
| **数字层 / FunMedia** | 非核心功能，优先级不高 |
| **Shift 做拇指键** | 需要把 Space 或 Alt 改造成 Shift，对现有布局是破坏性变更 |

---

## 紧急退出

如果配置出现严重问题导致键盘无法正常使用，同时按住以下 **物理键**（不受 Kanata 重映射影响）：

```
Left Ctrl + Space + Esc
```

Kanata 会立即退出，键盘恢复正常。

---

## 版本历史

| 版本 | 功能 | 触发方式 |
|:--|:--|:--|
| **v1.0.0** | CapsLock tap/hold + Nav Layer | CapsLock tap=Esc, hold=Nav |
| **v2.0.0** | + NavLock 导航层锁定（借鉴 Arsenik） | Nav 激活时按 R 锁定/解锁 |
| **v3.0.0** | + Symbols Layer 左手编程符号（借鉴 Arsenik） | 右 Alt hold → 左手 9 个符号 |

完整开发规格和踩坑记录请见 [kanata-开发文档.md](kanata-开发文档.md)。

---

## 致谢

- **Vim** — 让我爱上 HJKL 导航
- **那篇忘记标题的老文章** — 让我把 CapsLock 改成 Esc
- **[Kanata](https://github.com/jtroo/kanata)** — AHK v1 停维护后，让这个项目继续跑下去的优秀 Rust 重映射工具
- **[Arsenik](https://github.com/OneDeadKey/arsenik)** — NavLock 和左手符号层的灵感来源，"原来还能这样"的那一下
