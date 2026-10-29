# ARMCC · ARM Compiler 5.06

> 一套可直接投放使用的 ARM Compiler 5（AC5）文件集：`armcc`、`armasm`、`armlink`、`armar`、`fromelf` 与配套头文件、C/C++ 运行库。
>
> 主要用途：给 Keil MDK 5.37 及更高版本补回被官方移除的 AC5 编译器，让基于标准外设库的老工程重新可编译。

## 这是什么

ARM Compiler 5（缩写 AC5，命令行工具为 `armcc`）是 ARM 官方的经典 C/C++ 编译器，也是 Keil MDK 长期以来的默认编译工具。它面向 Cortex-M 系列 MCU，针对裸机与 RTOS 工程的代码体积、中断响应做过专门优化。STM32、GD32、NXP 等大量 MCU 的早期官方例程与第三方库，都是用它编译出来的。

这个仓库不是安装包，而是编译器安装目录的完整内容：可执行文件、标准库头文件、按内核架构与浮点选项切分的运行库、许可映射文件。放到正确位置即可被 Keil 或命令行直接调用，无需写注册表、无需联网激活。

## 为什么需要它

Keil MDK 从 5.37 版本起不再随安装包分发 AC5，只保留基于 Clang 的 AC6（`armclang`）。这带来一个很具体的问题：大量存量工程——尤其是依赖 STM32 标准外设库（StdPeriph）、旧版 CMSIS、2015 年前后创建的项目——在新版 Keil 中打开会直接报找不到编译器；强行切到 AC6 又会出现语法、内联汇编、启动文件不兼容等一连串告警。

官方仍提供 AC5 的独立下载，但入口分散、需要 ARM 账号、版本容易对不上。把 5.06 的整套文件归档在一处，能让同样被卡住的人少走几步。

## 目录结构

| 目录 | 内容 | 说明 |
| --- | --- | --- |
| `bin/` | `armcc.exe`、`armasm.exe`、`armlink.exe`、`armar.exe`、`fromelf.exe`、`armcompiler_libFNP.dll` | 编译器、汇编器、链接器、库管理器、镜像转换工具 |
| `include/` | 150 个头文件 | C 标准库、C++ 标准库（含 Rogue Wave 实现）及各架构相关头 |
| `lib/armlib/` | C 运行库 | 按内核架构、大小端、浮点 ABI、微库等维度提供多份变体 |
| `lib/cpplib/` | C++ 运行库 | 同上，另含 `cpprt_*` 运行时启动库 |
| `sw/` | `info/releasenotes.html`、第三方许可、`mappings/*.elmap` | 版本说明与许可映射文件 |

`lib` 下的 `.b` / `.l` 后缀分别对应大端与小端，同名文件成对出现。不要手动挑选单个库文件——链接器会依据 `--cpu` 与浮点选项自动匹配。

## 在 Keil MDK 中使用

1. 定位编译器目录，通常为 `C:\Keil_v5\ARM\ARMCC`。若不存在则手动新建。
2. 将本仓库的 `bin`、`include`、`lib`、`sw` 四个目录整体复制进该目录，保持层级不变。
3. 打开 Keil，进入 `Project → Manage → Project Items → Folders/Extensions`，确认 ARM Compiler 下拉框出现 `Use default compiler version 5` 并选中。
4. 老工程建议在 `Options for Target → Target` 中勾选 `Use MicroLIB`，与原始工程保持一致。

验证方式：新建空工程编译，`Build Output` 出现 `Toolchain: ARM Compiler 5` 且不再提示 `cannot execute armcc`，即为成功。

## 命令行直接使用

把 `bin` 加入 `PATH` 后，可以脱离 Keil 单独编译：

```bat
armcc --version
armcc -c --cpu Cortex-M3 -O2 --c99 -I .\include -o main.o main.c
armlink --cpu Cortex-M3 main.o --scatter .\stm32.sct -o app.axf
fromelf --bin app.axf --output app.bin
```

几个容易踩的点：

- 必须显式传 `--cpu`。缺省按 ARM7TDMI 生成，链到 Cortex-M 上会直接跑飞。
- C99 需要 `--c99`，AC5 不支持 C11 标准。
- 静态库路径需指向 `lib\armlib`；使用微库时改为 `--library_type=microlib` 并链接对应的 `*_p` 变体。

## 常见问题

**提示 `cannot execute armcc`，或编译器下拉里没有 V5。** 目录层级放错。`armcc.exe` 必须位于 `<Keil>\ARM\ARMCC\bin\`，多一层或少一层都会导致扫描不到。

**编译报找不到 `<stdint.h>` 之类的头文件。** 只复制了 `bin`。`include` 必须与 `bin` 同级一起复制。

**链接报 `undefined symbol`，涉及 `__use_no_semihosting` 等符号。** 老工程在 AC6 下常见，切回 AC5 后一般消失。若仍存在，检查是否混入了 AC6 编译的 `.o` 文件——两者目标文件不通用，需全量重编。

**路径含中文或空格导致异常。** AC5 是 2015 年前后的工具链，对非 ASCII 路径支持很差，工程路径建议全英文。

**关于许可。** `sw\mappings` 下的 `.elmap` 为许可映射文件。在 MDK 环境中随 Keil 授权使用；独立命令行调用需遵守 ARM 的授权条款。

## 版权说明

ARM Compiler 5 的著作权归 ARM Limited 所有。本仓库仅作个人学习与开发环境修复的归档，未对文件做任何修改或破解。商业项目请通过 ARM 官方渠道获取正式授权。若权利人认为不妥，可联系删除。

## 版本

ARM Compiler 5.06 系列。精确 build 号可用 `armcc --version` 查看。
