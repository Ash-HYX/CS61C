# Lab 8：虚拟内存（TLB 与页表）（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 8（Wayback 存档）整理翻译。英文原文见 `original-en/lab08.md`。
> 截止日期：EOD 11 月 6 日（周五）。

> **重要（2020 秋的 relief package）**：本 lab 只需到 TA 处 "check-in" 即可拿分。下面提到的 autograder 与 checkoff 问题都是**可选**的，但**强烈建议**完成——它们仍在期末考试范围内。

## 目标

- 探索虚拟内存的工作机制，特别是 TLB 与页表（Page Table）。
- 分析 TLB 命中率与页表命中率，并找出能优化它们的访问方式。

## 准备

```bash
git pull starter master
```

答案写在 `answers.txt`：

- 数值写数字，不要写英文单词（写 `7` 而不是 `seven`）。
- `Checkoff Question` 行会被 autograder 忽略，仅供你参考。
- 不要增删行、调换行或修改原有格式。

本 lab 主要使用 **Camera**（cache 与虚拟内存模拟器）的虚拟内存模拟功能。

> Camera 在 Hive/Linux 上常有兼容问题。建议下载 `Camera.jar` 到自己的（非 Linux）笔记本上双击运行；Mac 可能需要在"安全性与隐私"里点 "Open Anyway"。若内存里的数值挤在一起，改用 `CameraHDPI.jar`。实在跑不起来就和能跑的同学组队。

打开 Camera 后选择虚拟内存（virtual memory）选项：

- 左上：物理内存内容；
- 其下：本进程所有虚拟内存页；
- 右侧：TLB 与页表内容（初始为空）；
- 底部 "PROGRESS UPDATE"：内存系统统计与进度；
- 进度框右侧按钮：前进 / 后退 / 重头开始。

## 练习 0：Sanity Check

继续之前，**确认你打开的是 VM 模拟器，不是 cache 模拟器**。

## 练习 1：使用 Camera

点击右侧 "Auto Generate Add. Ref. Str."，生成 10 个地址引用（可理解为一系列 RISC-V `lw` 指令读取对应地址）。点 "Next" 开始模拟。

之后完全跟着 "PROGRESS UPDATE" 走：每点一次 "Next"，观察进度框与内存系统当前状态，真正理解每一步 TLB、页表、物理内存发生了什么。

模拟结束后记下 TLB 命中/缺失数、页命中/缺页（Page Fault）数，以及所用的访存序列，checkoff 时展示。

### 检查点

1. 按地址的划分方式，本模型中页有多大（以 word 为单位）？答案只写数字（如 `4`，不要写 "4 words"）。
2. 随机生成的 10 个地址中，TLB 命中/缺失各多少？页命中/缺页各多少？逗号分隔（如 `1, 2, 3, 4`）。
3. 是否出现过页命中？（为什么？）能否想出一组 10 个地址会产生页命中？答案是两个 `yes/no`，逗号分隔。

### Checkoff 问题

- 解释**第一次访问**时，虚拟地址如何转换为物理地址（重点：TLB Miss 与 Page Fault）。
- 为什么物理地址的 PPN 只有 2 位，而虚拟地址的 VPN 有 3 位？

## 练习 2：制造缺失

构造一组 10 次内存访问，使 TLB **缺失 10 次**、**缺页 10 次**。可以先在纸上设计，再用 Camera 验证。点 "Self Generate Add. Ref. Str." 逐条输入地址。

### 检查点

1. 写下这 10 次访存，格式为逗号分隔的十六进制值（如 `00, 01, 02, ...`）。

## 练习 3：减少缺页

对练习 2 的访存序列，能否只改变**一个参数**（如 TLB Size、Physical Memory Size、Virtual Memory Size、Page Size），使 TLB 仍缺失 10 次，但缺页**少于 10 次**？

### 检查点

1. 写出该参数（只改它、其他不变），使 TLB 缺失 10 次而缺页**更少**。

## 练习 4：融会贯通

用另一个模拟器 VMSIM 出题。二选一：

- 观看 VMSIM 模拟的 webm 视频；或
- 在终端运行 `appletviewer https://denninginstitute.com/workbenches/vmsim/vm.html`（Hive 上不可用）。

观察并回答：

- 本系统的设置与 Camera 有什么不同？特别是 P1、P2、P3、P4 是什么？
- 仔细看会发现该模拟的 TLB 缺失率比 Camera 随机运行高得多。为什么？
  （看不懂可用 `appletviewer`，把右下角速度条调慢。）

### Checkoff 问题

- 解释为什么这个模拟的 TLB 缺失率高得多。

## Checkoff

提交到 Gradescope 的 **Lab Autograder**（检查 `answers.txt`；忽略空白与大小写，但很简单，无法识别拼写/格式错误）。

checkoff 时准备讨论：

- 练习 1：第一次访问的地址转换过程（TLB Miss / Page Fault）；PPN 为什么只有 2 位而 VPN 有 3 位。
- 练习 4：为什么该模拟 TLB 缺失率高得多。
