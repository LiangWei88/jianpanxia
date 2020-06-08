# 键盘侠

基于 AutoHotKey 脚本的键盘增强工具

## Capslock 大写锁定键增强
大写锁定键基本上很少被使用, 这里将它设置为 Esc 键, 同时配合其他按键, 实现类似 vim 类似的浏览效果
原生 Capslock 锁定键改变为 Capslock + `

- **Caps** => Esc 按钮, 对于习惯 vim 使用者来说特别方便
- **Caps + `** => 原生 Caps 功能, 锁定/解锁大小写
- **Caps + h/j/k/l/w/b** => 类 vim 光标移动
    - h 左方向键
    - j 下方向键
    - k 上方向键
    - l 右方向键
    - w 下一个单词
    - b 上一个单词
- **Caps + u/i/o/p** => Home/End PageUp/PageDn
    - u 上一页
    - p 下一页
    - i Home
    - o End
- **Caps + n/m/,/.** => 退格/del 键
    - n 退格 backspace 删一个词
    - m 退格 backspace 删一个字符
    - , del 键 删一个字符
    - . del 键 删一个词