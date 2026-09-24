# Kanata Vim 风格全局导航键盘系统

> 本项目已从 **AutoHotkey v1** 实现迁移至 **Kanata**。
> AHK v1 官方已于 2024 年停止维护,为获得更好的性能、可移植性和配置体验,选择 Kanata 作为替代方案。

## 功能概述

正常情况下键盘行为完全保持原样,仅 **CapsLock** 键被特殊处理:

| CapsLock 行为 | 输出 |
|:--|:--|
| 短按 (tap) | `Esc` |
| 按住 (hold) | 激活 Nav Layer |

Nav Layer 下通过 Vim 风格按键完成方向移动、跳转、删除、浏览器前进后退等操作。松开 CapsLock 后立即恢复普通键盘行为。

## 为什么这样做?

这个项目的核心想法来自三个方向的灵感碰撞:

- **Vim 的 modal 哲学** — 模式切换让导航操作不需要离开主行
- **Vimium 的浏览器键盘驱动** — 证明了键盘导航不仅编辑器能用
- **Home Row Mods 的 tap/hold 区分** — 一个键同时承担两种功能,不增加按键数量

**本项目把三者合成为一个操作系统级的全局 Nav Layer** — 只要是能接收键盘输入的应用,都能用 Vim 风格按键导航,而且双手基本不离主行。

相比纯鼠标或依赖特定插件的方案,它的优势是:

- **全应用覆盖** — VS Code、浏览器、终端、文件管理器、设置面板统一生效
- **学习成本低** — 如果你用过 Vim/Vimium,HJKL、W/B、Y/O 一看就懂
- **不改变打字习惯** — Base Layer 完全保留,只有 CapsLock 被复用
- **Esc 补偿** — CapsLock 短按 = Esc,Vim 用户和终端重度使用者直接获益

> 更详细的设计背景、与 Vimium / HRM / 其他方案的对比、以及从 AHK v1 迁移到 Kanata 的原因,请看 [设计理念.md](设计理念.md)。

---

## 环境信息

| 项目 | 值 |
|:--|:--|
| Kanata 版本 | **1.12.0** |
| 操作系统 | Windows |
| 后端 | `winIOv2` (基于 LLHOOK + SendInput) |
| 配置文件 | `kanata.kbd` |

---

## 快速开始

### 启动 Kanata

在 `kanata.kbd` 所在目录打开 PowerShell 或 CMD,执行:

```powershell
.\kanata_windows_tty_winIOv2_cmd_allowed_x64.exe --cfg kanata.kbd
```

### 检查配置语法

修改配置后,建议先运行语法检查:

```powershell
.\kanata_windows_tty_winIOv2_cmd_allowed_x64.exe --check --cfg kanata.kbd
```

`--check` 通过后再正常启动,避免运行时出错。

---

## 键位表

### Base Layer

除 CapsLock 外所有键保持原始行为:

```
_ 1 2 3 4 5 6 7 8 9 0 - = bspc
tab q w e r t y u i o p [ ] \
@capnav a s d f g h j k l ; ' ret
lsft z x c v b n m , . / rsft
lctl lmet lalt   spc   ralt rmet rctl
```

- `@capnav` = CapsLock: tap → Esc / hold → Nav Layer

### Nav Layer (CapsLock 按住时激活)

| Caps + 键 | 输出 | 说明 |
|:--|:--|:--|
| H J K L | ← ↓ ↑ → | Vim 风格方向移动 |
| Y / O | Home / End | 行首 / 行尾 |
| U / I | PageUp / PageDown | 翻页 |
| W / B | Ctrl + Right / Left | 下一个 / 上一个单词 |
| G | Ctrl + Home | 文档开头 |
| Shift + G | Ctrl + End | 文档末尾 |
| X | Delete | 向后删除 |
| C | CapsLock | 保留原切换功能 |
| `;` / `'` | Alt + Left / Right | 浏览器 / 文件管理器后退前进 |
| `,` / M | 鼠标滚轮向上 / 向下 | 带惯性加速 |

> Nav Layer 中未列出的按键均透明传递,保持 Base Layer 的原始行为。

### 鼠标滚轮惯性参数

| 参数 | 当前值 | 说明 |
|:--|:--|:--|
| initial-velocity | 3 | 初始速度 |
| max-velocity | 1200 | 最大速度 |
| acceleration | 1.15 | 按住时加速度 |
| deceleration | 0.93 | 松开后衰减系数 |

---

## 可调参数

所有参数集中定义在 `kanata.kbd` 的 `defvar` 区块:

```scheme
(defvar
  tap-timeout   180   ;; CapsLock tap 判定超时 (毫秒)
  hold-timeout  180   ;; CapsLock hold 激活超时 (毫秒)
)
```

### 调整 CapsLock tap/hold 时间

修改 `tap-timeout` 和 `hold-timeout` 的值即可。两值相等时体验最稳定。推荐测试值:

```
160 / 180 / 200 / 220 (毫秒)
```

- 值越小 → 响应越快,但更容易误判
- 值越大 → 容错越高,但感觉稍慢

修改后执行 `--check` 确认语法正确,再重启 Kanata。

---

## 配置结构

```
defcfg       → 全局配置 (并发 tap-hold、未映射键处理)
defsrc       → 定义被 Kanata 拦截的物理键
defvar       → 可调参数 (tap/hold 超时时间)
defalias     → 别名定义 (Caps tap/hold、组合键、滚轮)
deflayer base → Base Layer (原始键位 + CapsLock 特殊处理)
deflayer nav  → Nav Layer (CapsLock 按住时激活)
```

---

## Windows 注意事项

### winIOv2 vs Interception

本项目使用 `winIOv2` 后端,基于 Windows 原生 `SetWindowsHookEx` (LLHOOK) + `SendInput`,无需安装第三方驱动。

如果后续遇到某些应用(如游戏、反作弊)中按键不生效,可考虑切换至 `interception` 后端(需额外安装驱动)。

### 以管理员身份运行

部分高权限应用(如以管理员运行的终端)可能无法被 LLHOOK 拦截。建议 Kanata 也以管理员身份运行。

### AltGr 兼容

如果键盘布局使用 AltGr,可能遇到 AltGr 行为异常。可在 `defcfg` 中加入:

```scheme
windows-altgr cancel-lctl-press
```

---

## 紧急退出

如果配置出现严重问题导致键盘无法正常使用,同时按住以下 **物理键** (不受 Kanata 重映射影响):

```
Left Ctrl + Space + Esc
```

Kanata 会立即退出,键盘恢复正常。

---

## 配置验证清单

修改 `kanata.kbd` 后,按以下步骤验证:

1. `--check` 语法检查通过
2. Base Layer: A-Z、数字、符号、Enter、Backspace、Tab、修饰键行为不变
3. Caps tap → Esc, Caps hold → Nav Layer
4. Nav Layer 各键位符合预期
5. 快速组合输入 (W/B/X 等) 不出现 tap/hold 误判
6. 连续按住方向键可重复触发
7. VS Code、Terminal、浏览器、文件管理器中功能正常
