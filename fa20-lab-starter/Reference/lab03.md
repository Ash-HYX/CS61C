# Lab 3：RISC-V 汇编与 Venus 模拟器（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 3（Wayback 存档）整理翻译。英文原文见 `original-en/lab03.md`。
> 截止日期：9 月 18 日（周五）。本 lab 比前后几个都长，建议尽早开始。

## 目标

- 练习运行与调试 RISC-V 汇编代码。
- 按正确的函数调用约定编写 RISC-V 函数。
- 了解如何把 C 代码翻译成 RISC-V。
- 熟悉 Venus 模拟器。

## 准备文件

```bash
$ git pull starter master
# 若报 starter 不是 git 仓库：
$ git remote add starter https://github.com/61c-teach/fa20-lab-starter.git
$ git pull starter master
```

## 汇编与 Venus 基础

C 程序由 `gcc` 编译成机器码直接执行；RISC-V 汇编（`.s`）更接近机器码，但你的电脑/Hive 跑的是 x86 或 ARM，所以需要 **Venus** 模拟器（网页版 https://venus.cs61c.org，仓库 `tools/` 目录也有 `venus.jar`）。

先阅读 Venus reference 的 "The Editor Tab" 和 "The Simulator Tab" 两节。

> **务必把完成的代码保存成本地文件**，否则无法证明你完成了实验。

## 练习 0：把本地文件挂载到 Venus

新版 Venus 支持把本地文件夹"挂载"到网页端，编辑结果会同步到本地文件。

```bash
# 在 labs 目录下运行，把当前目录暴露到网络端口（默认 6161）
$ java -jar tools/venus.jar . -dm
```

- 看到提示 `To connect, enter \`mount http://localhost:6161 vmfs\` on Venus.` 即成功。
- 若提示端口无法绑定，可显式指定：`java -jar tools/venus.jar . -dm --port 6162`。

然后：

1. 浏览器打开 https://venus.cs61c.org ，在 Venus 终端执行 `mount local vmfs`（改过端口则用完整 URL，如 `mount http://localhost:6162 vmfs`）。
2. 打开 "Files" 标签，应能看到 `vmfs` 下的 labs 目录。
3. 进入 `lab03`，点 `ex1.s` 旁的 `Edit`，文件会在 Editor 标签打开。
4. 在 Editor 中改文件后按 `Cmd+S`（Mac）/ `Ctrl+S`（Win/Linux）保存到本地；暂无保存成功提示，可打开本地文件确认。
5. 本地修改过的文件若在 Venus 中已打开，需要从 "Files" 菜单重新打开才会看到新内容。
6. 可在 Settings 里勾选 "Save on Close"，让 Venus 重开时自动重挂载；若服务未运行会报连接失败，按上面步骤手动重挂即可。

> 如果挂载功能不可用（较新、可能有 bug）：把文件内容复制粘贴进 Venus 编辑器，再把改动粘回本地。

## 练习 1：熟悉 Venus

1. 在 Editor 打开 `ex1.s`。
2. 切到 "Simulator"，点 "Assemble & Simulate from Editor"。切回 Editor 会重置模拟。
3. `step`：执行下一条指令；`prev`：撤销上一条（对 `ecall` 的效果不一定能撤销）。
4. `run`：运行到结束；`reset`：从头开始。
5. 右侧显示 32 个寄存器的内容，底部是控制台输出。
6. 右侧 "Memory" 标签可查看内存，底部下拉菜单可跳转不同区域。

### 任务（把答案记录下来）

1. `.data`、`.word`、`.text` 伪指令分别是什么意思？（提示：内存的四个段。）
2. 运行完程序，输出什么数字？它代表什么？
3. 变量 `n` 存在内存的哪个地址？（提示：看寄存器内容。）
4. **不修改代码**（不进 Editor），通过**手动修改某个寄存器的值**，让程序计算第 13 个斐波那契数（0-indexed）。可先单步走一遍；想看十进制可改底部 "Display Settings"。

## 练习 2：C 到 RISC-V 的翻译

打开 `ex2.c` 与 `ex2.s`（`.s` 是 C 程序的 RISC-V 翻译）。除了在 Editor/Simulator 中运行，也可在 Venus 终端 `cd` 到目录后 `run ex2.s` 或 `./ex2.s`；`vdb ex2.s` 会汇编并直接跳到 Simulator。

### 任务：找出并解释

- 表示变量 `k` 的寄存器；
- 表示变量 `sum` 的寄存器；
- 作为 `source`、`dest` 数组指针的寄存器；
- C 中循环对应的汇编代码；
- 汇编中指针是如何被修改的。

## 练习 3：阶乘

在 `factorial.s` 中实现 `factorial(n)`，返回 `n!`。

- 只需在 `factorial` 标签下添加指令；参数位于标签 `n` 处。
- 递归或迭代均可。
- 假设只传入正数，且结果不溢出 32 位补码。

自检：`3! = 6`、`7! = 5040`、`8! = 40320`。

本地测试（autograder 也用 jar，需安装 Java）：

```bash
$ java -jar tools/venus.jar lab03/factorial.s
```

## 练习 4：调用约定检查器（CC checker）

`cc_test.s`。回顾调用约定：所有会覆盖"被调用者保存寄存器"的函数都必须有 **prologue**（入口把相关寄存器值压栈保存）和 **epilogue**（返回前恢复）。详见课程资源 `RISCV_Calling_Convention.pdf`。

```bash
$ java -jar tools/venus.jar -cc lab03/cc_test.s
```

> `-cc` **必须**放在 `venus.jar` 与文件名之间。放在 `venus.jar` 前会被当成 java 的参数；放在文件名后会被当成汇编程序 `main` 的参数。

CC checker 会报告违规，例如：

```
[CC Violation]: (PC=0x00000080) Usage of unset register t0! main.S:58 mv a0, t0
[CC Violation]: (PC=0x0000008C) Setting of a saved register (s0) which has not been saved! main.S:80 li s0, 1
[CC Violation]: (PC=0x000000A4) Save register s0 not correctly restored before return! Expected 0x00000A3F, Actual 0x00000080. main.S:90 ret
...
Venus ran into a simulator error!
Attempting to access uninitialized memory between the stack and heap.
```

输出的 `main.S` 只是 Venus 支持多文件时的显示产物，行号仍对应 `cc_test.s`。所有 CC 错误信息及含义见 Venus reference 的 "Calling Convention Checker"。

- 修完 CC checker 报的错后程序**可能仍然失败**：还有它检测不到的调用约定错误（汇编里函数调用本质是跳转，Venus 缺少信息、怕误报）。
- 所有修复都加在 starter 代码中 `FIXME` 注释标记的行附近。
- CC checker 只检查 `.globl` 导出的函数，且只是"辅助自检"，不能保证查出所有问题。

### 任务

修好 `cc_test.s` 中所有调用约定错误，并能回答：

- CC checker 在 `simple_fn`、`naive_pow`、`inc_arr` 中报的错分别由什么引起？
- RISC-V 通过跳转调用函数、把返回地址存到 `ra`。跳到 `naive_pow_loop`、`naive_pow_end` 这类**标签**时，调用约定适用吗？
- 为什么 `inc_arr` 的 prologue 需要保存 `ra`，而其他函数不需要？
- 为什么 `helper_fn` 中的调用约定错误没有被 CC checker 报出来？（提示见上：只检查 `.globl` 导出的函数。）

然后对上个练习的 `factorial.s` 也跑一遍 `-cc`，修掉发现的问题。

### 测试

修完后运行上面的命令，行为不应改变，且输出：

```
Sanity checks passed! Make sure there are no CC violations.
Found 0 warnings!
```

## 练习 5：用 `map` 练习 RISC-V 函数调用

文件 `list_map.s`。在单链表上实现 [`map`](https://en.wikipedia.org/wiki/Map_(higher-order_function))，**原地修改**链表（不新建链表）。可参考 RISC-V green card（课程 resources 的 `riscvcard.pdf`）。

数据结构：

```c
struct node {
    int value;
    struct node *next;
};
```

`map` 接收两个参数：链表头节点地址；以及一个**函数地址**（`int f(int)`）。用 `jalr` 调用该函数。C 版本：

```c
void map(struct node *head, int (*f)(int)) {
    if (!head) return;
    head->value = f(head->value);
    map(head->next, f);
}
```

代码中共 **9 处** `YOUR CODE HERE`（`map` 中 8 处，`main` 中 1 处）。

### 任务

- 填完 9 处标记（每处上方的注释说明了要做什么）；
- 在 `main` 中给出以 `square` 为函数参数的示例调用。

正确输出：

```
9 8 7 6 5 4 3 2 1 0
81 64 49 36 25 16 9 4 1 0
```

第一行是原链表，第二行是应用 `square` 后的结果。

### 测试

网页版直接运行 `list_map.s` 看输出；本地：

```bash
$ java -jar tools/venus.jar lab03/list_map.s
```

## 后续：更复杂的 RISC-V 程序

之后会用到多文件汇编，建议提前阅读 Venus reference 的：

- Writing Larger RISC-V Programs
- Using the Web Terminal
- Passing Command Line Arguments

## Checkoff

提交到 Gradescope 的 **Lab Autograder**。

checkoff 时要能回答练习 1、2、4 中的问题。
