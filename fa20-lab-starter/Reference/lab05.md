# Lab 5：Logisim 数字逻辑（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 5（Wayback 存档）整理翻译。英文原文见 `original-en/lab05.md`。
> 截止日期：10 月 7 日（周三）～10 月 9 日（周五）之间 checkoff。

> **重要：不要移动助教提供的输入/输出引脚**，否则测试框架/harness 会失效。若电路自认为正确却测不过，检查它是否能放进 `testing/circ_files/exX_test.circ` 的对应 harness。

## 目标

- 在 Logisim 中练习设计与调试基本数字逻辑电路。
- 练习组合逻辑 + 状态元件（寄存器）的电路。
- 设计 FSM 并用数字逻辑实现。

## 准备

```bash
git pull starter master
```

本 lab 全部使用 starter 附带的 **Logisim Evolution**：

```bash
java -jar logisim-evolution.jar
```

> - **必须用我们给的 .jar**，不要用实验机上自带的 Logisim。
> - Logisim **不会自动保存**，打开时也**不会自动新建 .circ**——开始时就保存，并经常保存。
> - Logisim 需要图形界面：本地做最方便；远程需 `ssh -X`（Windows 还要 Xming，macOS 要 XQuartz）。能本地跑就别远程跑。

课程还提供了可选的介绍视频（覆盖练习 0~3 的 Logisim 基础操作），但各练习的 action item 仍要完成。

## 入门

### Part 0：热身（放门电路和连线）

1. 点 `AND` 门按钮，在主绘图区点一下放置。
2. 点 `Input Pin`，在 AND 门左侧放两个输入引脚。
3. 点 `Output Pin`，在右侧放一个输出引脚。
4. 用 `Select tool` 拖拽连线：只能画水平/垂直线，先水平拖、松开，再从线端继续垂直拖。把两个输入接到 AND 左侧，AND 右侧接到输出。
5. 用 `Poke` 工具点输入引脚，观察输出是否符合 AND 的预期（点导线本身可查看该线当前值，后面很有用）。

左下角的**缩放**功能非常有用。

### Part 1：子电路（Sub-circuits）

> Logisim Evolution 规定：子电路不能与关键字同名（如 `NAND`），且名字必须以 `A-Za-z` 开头，不能以数字开头。

#### 任务（完成后向 TA 展示，记得保存）

1. 打开练习 1 电路：`File -> Open -> ex1.circ`。
2. 在左侧电路选择器中双击 `NAND1`（末尾有 1，因为已有名为 `NAND` 的元件），进入空的子电路。
3. 只用 `AND`、`OR`、`NOT`（**不许用内置 NAND 门**）搭出 NAND：两个输入在左，输出在右。可用 select 工具选中引脚，在左下属性里改 `Label`。
4. 同样在给定骨架中完成 `NOR`、`XOR`、`2-to-1 MUX`、`4-to-1 MUX`。**不要改子电路名、不要新建**，否则 autograder 失效；除 `AND/OR/NOT` 外不得使用其他内置门。建好的子电路可像普通元件一样复用。`4-to-1 MUX` 中 `Sel0`、`Sel1` 分别是 2 位选择信号的第 0、1 位。提示：写真值表；可复用自己搭的子电路。

### Part 2：存储状态（寄存器）

做一个"无限自增"的电路——与之前电路的区别是把值**存进寄存器的状态**里。

#### 任务（完成后展示）

1. 打开 `ex2.circ`，进入空的 `AddMachine` 电路。
2. 载入 `Arithmetic` 库：`Project -> Load Library -> Built -> Arithmetic`（左侧出现 Arithmetic 文件夹），放置加法器。
3. 载入 `Memory` 库（同上），放置一个寄存器。
4. 把 `clock`（在 `Wiring` 文件夹）接到寄存器。
5. 加法器输出接寄存器输入，寄存器输出接加法器输入。
   - 若报 "Incompatible widths"：选中加法器，把左下属性 `Data Bit Width` 改成 **8**。
6. 用 `Wiring` 里的 `Constant` 给加法器另一输入接 8 位常数 `1`。
7. 加法器输出接 `ADD_OUT`，寄存器输出接 `REG_OUT`。
8. `Simulate -> Ticks Enabled`（`Ctrl/Cmd + K`）开始运行，电路应输出二进制计数器。可用 `Simulate -> Tick Frequency` 调快慢。

### Part 3：FSM 转数字逻辑

FSM（有限状态机）记录已收到的输入、按输入在状态间转移、并在每次输入时产生输出。用寄存器存"当前状态"，用组合逻辑把"输入 + 当前状态"映射到"输出 + 下一状态"。

#### 任务

打开 `ex3.circ`，修改子电路 `StateBitZero`、`StateBitOne`，实现：

> **若曾出现过连续两个 1 或连续两个 0，则此后永远输出 0；否则输出 1。**

状态图中的状态名 `00`、`01`、`10`、`11` 只是二进制编号，与 FSM 的输入/输出值没有直接关系。给定真值表：

| st1 | st0 | input | next st1 | next st0 | output |
|-----|-----|-------|----------|----------|--------|
| 0 | 0 | 0 | 0 | 1 | 1 |
| 0 | 0 | 1 | 1 | 0 | 1 |
| 0 | 1 | 0 | 1 | 1 | 0 |
| 0 | 1 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 0 | 1 | 1 |
| 1 | 0 | 1 | 1 | 1 | 0 |
| 1 | 1 | 0 | 1 | 1 | 0 |
| 1 | 1 | 1 | 1 | 1 | 0 |

顶层电路与之前的加法器电路几乎一样，只是加法器换成了 `FSMLogic` 组合逻辑块。输出位已帮你实现（它最难化简），你只需完成产生**下一状态位**的 `StateBitOne`、`StateBitZero`。

## Logisim 进阶

三件省时又让电路更整洁的工具：

- **Splitter（分线器）**：把多位总线拆成多条低位宽线，或反向合并。点选后在侧栏设置 `Bit Width In`、`Fan Out`、每个 arm 的位、`Facing` 方向。若导线变**橙色**，表示两端位宽不匹配。
- **Tunnel（隧道）**：用同名标签（区分大小写）连接"看不见的线"，让电路更清爽、更好调试。
- **Extender（位扩展器）**：显式改变线宽（8 位 → 16 位，或丢弃高位 8 位 → 4 位），比手动拆线更清晰、更不易错。

### Part 4：Splitter 练习

在 `ex4.circ` 的空 `Split` 电路中：

1. 放 `Splitter`（`Wiring` 文件夹），`Bit Width In = 8`、`Fan Out = 3`。
2. 分配位：bit 0 → arm 0；bit 1~6 → arm 1；bit 7 → arm 2（`None` 表示该位不出现在任何 arm）。
3. `In1` 接 splitter；arm 0 与 arm 2 接 2 输入 `AND`，输出接 `Out1`。
4. 把输入解释为"符号-数值（sign and magnitude）"表示，用逻辑门让 `Out2` 输出其相反数（符号-数值表示比补码更简单，组合逻辑很直接）。
5. 再用一个 splitter（`Bit Width In: 8`, `Fan Out: 3`, 正确 arm 位宽）把各路合回 8 位总线。调整 `Facing`、`Appearance` 让电路整洁。此时 `Out2` 应为输入的相反数（按符号-数值解释）。
6. 回答问题：**若把输入解释为补码，哪些输入会使 `Out1 = 1`？**（提示：补码的首位和末位都为 1 说明了什么？）

### Part 5：循环右移（rotr）

实现组合逻辑块 `rotr`：把 16 位输入 `A` 循环右移 `B` 位。例：`A=0b1011010101110011`、`B=0b0101`（5），结果为 `0b1001110110101011`——最右边 5 位被移到最左边。RTL 写法类似 `R = A >> B | A << (16 - B)`。

#### 任务

在 `ex5.circ` 中实现子电路 `rotr`：

- `A`：16 位，待旋转的值；
- `B`：4 位，旋转量（想想为什么是 4 位？）；
- 输出：`A` 循环右移 `B` 位。

限制：**不得使用 Logisim 的移位器（shifter）**；其他组合逻辑（MUX、常数、门、加法器等）可用。`Plexers` 里的内置 MUX 会很有帮助。不能使用时钟或寄存器等时序元件。

- 提示 1：先想清楚如何把问题拆成小块再组合；**放心使用子电路**，否则会后悔。
- 提示 2：RTL 写法未必是最好的视角。想想 `B` 的各个二进制位，以及如何用 splitter：`9 = 1001 = 1*8 + 0*4 + 0*2 + 1*1`。可以利用题目提供的 `rot*` 子电路（如 rot1、rot2、rot4、rot8）串起来，电路会很干净。

## 测试

```bash
./test.sh
```

Logisim 已占一个终端窗口，请另开窗口运行；若提示无权限：

```bash
chmod +x test.sh
```

脚本会把你的电路复制进测试 harness，用不同输入运行并与参考答案比较。**不要动 `testing` 文件夹里的任何东西**（除非 TA 要求）；但可以阅读其中的测试电路，Project 3 还会遇到。

## Checkoff

提交到 Gradescope 的 **Lab Autograder**。

向 TA 展示并解释以下电路：`2-to-1 MUX`、`4-to-1 MUX`、`AddMachine`、`StateBitZero`、`StateBitOne`、`Split`、`rotr`。
