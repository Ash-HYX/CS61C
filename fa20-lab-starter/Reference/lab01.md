# Lab 1：C 语言与 CGDB 调试（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 1（cs61c.org/fa20/labs/lab01，Wayback 存档）整理翻译。英文原文见 `original-en/lab01.md`。
> 截止日期：9 月 4 日（周五）。

## 学习目标

- 在教学机器上编译、运行 C 程序，观察 C 的几种控制流结构。
- 初步使用 C 调试器（gdb/cgdb）调试程序。
- 用整数、字符、布尔表达式、位运算在 C 中模拟正则表达式。

## 准备

```bash
$ git pull starter master
# 若报 fatal: 'starter' does not appear to be a git repository：
$ git remote add starter https://github.com/61c-teach/fa20-lab-starter.git
$ git pull starter master
```

本地安装（可选）：

- macOS：`xcode-select --install`，用 `gcc --version` 验证。
- Ubuntu/Debian：`sudo apt install build-essential cgdb valgrind`。
- Windows：建议用 WSL，参见 Piazza 的 "Setup for Windows Users"。

## 编译与运行 C 程序

```bash
$ gcc program.c            # 生成 a.out
$ ./a.out                  # ./ 表示当前目录，用于区分可执行文件与系统命令
$ gcc -o program program.c # 用 -o 指定可执行文件名
$ ./program
```

## 练习 1：See what you can C

宏定义在编译前做纯文本替换：

```c
#define CONSTANT_NAME literal_value
```

打开 `eccentric.c`，注意其中四种基本 C 控制流示例（想想分别是哪四种）。先编译运行，再分别修改 `V0`~`V3` 四个宏的值，观察输出变化。

### 任务

只修改这四个宏的值，让程序输出：

```
Berkeley eccentrics:
====================
Happy Happy Happy
Yoshua
Go BEARS!
```

挑战：在仍能得到该输出的前提下，`V0`~`V3` 最少需要多少个**互不相同**的值？（理论最大值是 4，即四者互异。）

编译时可执行文件请命名为 `eccentric`（用 `-o`）。

## 练习 2：Catch those bugs!

调试器（debugger）用于定位逻辑错误。所有调试器通常都能：

1. 设置**断点（breakpoint）**——在指定行暂停，便于查看现场。
2. **逐行执行**——代码本质上是逐行执行的，只是太快，逐行执行才能定位问题。

参考 GDB 速查卡（课程 resources 里的 gdb5-refcard.pdf）。GDB = GNU De-Bugger。

```bash
$ gcc -g -o hello hello.c   # -g 把调试信息写进可执行文件
$ cgdb hello                # 对可执行文件运行调试器，不要对 hello.c 运行
# 若 cgdb 不可用：gdb hello
```

> cgdb 只装在 `cs61c-xxx` 账号上（为 Ubuntu 编译），请在 Hive 或 Soda 27x 机器上使用。本地 macOS 新版可能装不上，可改用 lldb（命令略有差异）。

### 任务 1

完整地单步走一遍程序：

1. 在 `main` 处设断点；
2. 用 run 命令运行；
3. 用单步命令逐行执行。

在 gdb 里输入 `help` 或查速查卡找命令。

**若看到 `printf.c: No such file or directory`**：你步入（step into）了 `printf` 内部。用 `finish` 让当前函数返回；以后用 `next` 跳过函数调用行。

### cgdb 与 gdb

cgdb 与 gdb 命令完全一致，只是多了分屏界面。`ESC` 进入上方代码窗口，`i` 回到下方命令窗口（类似 vim）。

### 任务 2：`gdb.txt`

在 `gdb.txt` 中回答（每题以编号开头，如 `1. answer`；**只写命令名**，不要写参数）：

1. 在 gdb 会话中，如何设置程序运行时传入的**命令行参数**？
2. 如何**创建断点**？
3. 停在断点后，如何**执行下一行 C 代码**？
4. 若下一行是函数调用，按第 3 题的答案会一次性执行完整个函数。如何让 GDB **进入函数内部**调试（step into）？
5. 停在断点后如何**继续运行**？
6. 如何**打印变量（或 `1+2` 这样的表达式）的值**？
7. 如何让 gdb 在**每一步之后都自动显示某变量的值**？
8. 如何**列出当前函数所有变量及其值**？
9. 如何**退出** gdb？

> 参考答案（做完再对）：`run`、`break`、`next`、`step`、`continue`、`print`、`display`、`info locals`、`quit`。

## 练习 3：带用户输入的调试

```bash
$ gcc -g -o int_hello interactive_hello.c
$ ./int_hello
$ cgdb int_hello
```

`interactive_hello.c` 用 `fgets` 从 `stdin` 读输入。在 CGDB 里直接 run 会卡在等待输入处。
办法是**重定向**：把输入写进文件，再在 CGDB 里带重定向符号运行。

```bash
$ echo "your input" > input.txt
# 在 cgdb/gdb 中：
run < input.txt
# 命令行直接跑也可以：
$ ./int_hello < input.txt
```

提示：可以创建包含输入的文本文件（提示 1）；CGDB 的 run 也支持命令行参数与重定向符号（提示 2）。
目的：让你不再害怕调试需要输入的程序——后面的 project 会经常用到。

## 练习 4：Valgrind 排雷

- **bohrbug（玻尔 bug）**：在确定（但可能未知）条件下稳定复现——调试器可抓。
- **heisenbug（海森堡 bug）**：一研究就消失/改变行为，C 中常源于内存管理错误——调试器可能漏掉。

C 需要程序员手动管理内存。Valgrind 模拟 CPU 并跟踪内存访问：程序会变慢，但能暴露隐藏 bug。

```bash
$ gcc -g -o segfault_ex segfault_ex.c
$ gcc -g -o no_segfault_ex no_segfault_ex.c
$ ./segfault_ex        # 段错误
$ ./no_segfault_ex     # 不崩，但仍有问题
$ valgrind ./segfault_ex
$ valgrind ./no_segfault_ex
```

要点：

- **segfault**：程序访问了不属于它的内存（早期"分段内存"的遗留说法，今天用分页）。打开 `segfault_ex.c` 看原因但**不要改文件**；Valgrind 会指出非法访问发生的位置——以后在大文件里定位 segfault 就靠它。
- `no_segfault_ex` 不崩，但使用了**未初始化**的值，Valgrind 会报 undefined behaviour，虽不直接指出行号，但能据此推断哪个变量出了问题。

思考：

- Valgrind 为什么重要、有什么用？
- 如何在 Valgrind 中运行程序？
- 如何解读错误信息？（别怕，先自己分析再问助教。）
- 为什么未初始化变量会造成 heisenbug？

学完本周 C 内存模型后再回来回答：

- `no_segfault_ex` 为什么**没有**段错误？
- 为什么它输出不一致？
- `sizeof` 为什么用错了？怎样仍用 `sizeof` 但改对代码？

## 练习 5：C 中的指针与结构体（`ll_has_cycle`）

在 `ll_cycle.c` 中实现判断单链表是否有环的"龟兔赛跑"（Floyd）算法：

1. 两个指针都指向 head：`tortoise` 与 `hare`。
2. `hare` 前进两个节点；若因空指针走不了，说明到表尾，无环。
3. `tortoise` 前进一个节点（为什么不需要判空？）。
4. 若二者指向同一节点，则有环；否则回到第 2 步。

`node` 结构体定义见 `ll_cycle.h`。

### 任务

```bash
$ gcc -g -o test_ll_cycle test_ll_cycle.c ll_cycle.c
$ ./test_ll_cycle
```

只要改了代码，就按顺序重跑**全部**上面两条命令。

参考实现：

```c
int ll_has_cycle(node *head) {
    node *tortoise = head, *hare = head;
    while (hare != NULL && hare->next != NULL) {
        hare = hare->next->next;
        tortoise = tortoise->next;
        if (tortoise == hare) return 1;
    }
    return 0;
}
```

> 两种常见写法差别只在终止条件：一种要在开头处理特殊情况，另一种多加几次 NULL 检查——都可以，不必纠结"优雅"。算法原理见 Wikipedia "Cycle detection / Floyd's Tortoise and Hare"，不会考。

## Checkoff 与提交

提交到 Gradescope 的 **Lab Autograder**（与上周相同）。gdb 答案的 autograder 很简单，注意：

1. `gdb.txt` 每一行都以题号开头（如 `1. answer`）。
2. 只写**命令名**：如命令是 `command arg`，只写 `command`。

checkoff 前准备好解释你做了什么、改了什么。
