# CS61C Fall 2020 Labs 中文参考（Reference）

本文件夹是对 **UC Berkeley CS61C（Great Ideas in Computer Architecture）Fall 2020** 实验说明的中文整理，配合本仓库（`61c-teach/fa20-lab-starter`）使用。

- `lab00.md` ~ `lab11.md`：每次实验的中文说明（目标、步骤、命令、要交的文件、checkoff 要点）。
- `original-en/lab00.md` ~ `original-en/lab11.md`：官方英文原文存档，便于中英对照。

> 说明：2020 年秋季课程网站 `cs61c.org/fa20/labs/labXX/` 已下线，本文件夹中的英文原文取自 **Wayback Machine（web.archive.org）** 的存档页面，并转换为 Markdown；中文部分据此整理翻译，保留了练习编号、参数与命令，属于"导读 + 要点翻译"，不是逐字对照译本。原文中的失效链接（Google 表单、Gradescope、Piazza、Hive 等）属于历史信息，自学时无需访问。

## 实验一览

| Lab | 主题 | 截止时间 | 主要内容 | 提交/验收 |
|---|---|---|---|---|
| [Lab 0](lab00.md) | 课程介绍与环境搭建 | 9/11（原 9/4，仅本 lab 延长一周） | 课程政策、GitHub/Gradescope/Hive/Piazza、SSH/scp/Vim、Git 练习（合并冲突） | autograder + checkoff |
| [Lab 1](lab01.md) | C 与 CGDB | 9/4 | gcc、宏、cgdb/gdb 调试、stdin 重定向、Valgrind、链表判环 | autograder + checkoff |
| [Lab 2](lab02.md) | 进阶 C、Valgrind | 9/11 | Makefile、位运算、LFSR、动态数组 `vector` 与内存管理 | autograder + checkoff |
| [Lab 3](lab03.md) | RISC-V 与 Venus | 9/18 | Venus 文件挂载、汇编基础、C→RISC-V、阶乘、调用约定检查器、链表 `map` | autograder + checkoff |
| [Lab 4](lab04.md) | RISC-V 函数与指针 | 9/25 | 调试 `megalistmanips.s`（找 5 处错误）、无分支实现离散函数 | autograder + checkoff |
| [Lab 5](lab05.md) | Logisim 数字逻辑 | 10/7~10/9 checkoff | 门电路、子电路、MUX、寄存器、FSM、splitter、`rotr` | autograder + checkoff |
| [Lab 6](lab06.md) | 流水线与时钟频率 | 10/16 | 关键路径与最高时钟频率、乘法/加法两级流水线、气泡 | 仅 checkoff |
| [Lab 7](lab07.md) | Cache | 10/30 | Venus cache 可视化、命中率场景、矩阵乘法循环顺序、cache blocking 转置 | autograder + checkoff |
| [Lab 8](lab08.md) | 虚拟内存（TLB/页表） | 11/6 | Camera 模拟器、TLB/页表/缺页、构造访存序列、VMSIM | autograder + checkoff（当年仅需 check-in） |
| [Lab 9](lab09.md) | SIMD 指令 | 11/13 | SSE intrinsics、`sum_simd`、循环展开、性能对比 | autograder + checkoff |
| [Lab 10](lab10.md) | 线程级并行 | 11/20 | OpenMP hello、向量加法（slicing/chunking）、点积（critical/reduction）、多进程 HTTP 服务器 | autograder + checkoff |
| [Lab 11](lab11.md) | Spark 与 MapReduce | 12/4 | Spark 环境、word count、文档词频、全文索引、最流行词 | autograder + checkoff |

## 建议使用方式

1. **先读中文说明**，明确本次实验要交什么、checkoff 会问什么。
2. 打开对应 `labXX/` 目录里的 starter 文件动手；需要精确措辞或细节时对照 `original-en/labXX.md`。
3. 注意各 lab 的通用约定：
   - 起始代码更新：`git pull starter master`；首次需
     `git remote add starter https://github.com/61c-teach/fa20-lab-starter.git`。
   - 需要自动评分的 lab，把仓库 push 后在 Gradescope 提交 `Lab Autograder`。
   - 答案文件（`answers.txt`、`exercise*.txt`、`gdb.txt`、`make.txt`）不要改动原有格式，数值写数字而不是英文单词。
   - 2020 秋的 lab 要求两人一组 checkoff（Lab 8 因 relief package 只需 check-in）。
4. 环境提示：原课程依赖 Hive 教学机（`hive1`~`hive30`）与 `cs61c-???` 账号；这些服务现已不可用。自学时可在本地 Linux（或 WSL）安装 `gcc`、`cgdb`、`valgrind`、`openmp`，用 `java -jar tools/venus.jar` 跑 Venus，Logisim/Camera 用 starter 里的 jar。

## 常用命令速查

```bash
# 取出/更新起始代码
git pull starter master

# C 编译与调试
gcc -g -o prog prog.c
cgdb prog                  # 或 gdb prog
valgrind --leak-check=full --track-origins=yes ./prog

# RISC-V（Venus）
java -jar tools/venus.jar lab03/factorial.s
java -jar tools/venus.jar -cc lab03/cc_test.s     # 调用约定检查
java -jar tools/venus.jar . -dm                   # 把当前目录挂载到网页版 Venus

# Logisim
java -jar logisim-evolution.jar

# OpenMP
gcc -fopenmp -o prog prog.c
OMP_NUM_THREADS=4 ./prog
```

## 社区中文资料（可选，学期可能不同）

以下是搜索到的中文笔记/翻译，可作补充；其对应的学期、练习编号可能与 fa20 不完全一致，请以本文件夹和英文原文为准：

- 知乎《【CS61C】Lab 1: C and GDB》<https://zhuanlan.zhihu.com/p/640482035>
- 知乎《【CS61C】Lab 2: Advanced C, Valgrind》<https://zhuanlan.zhihu.com/p/640482564>
- 知乎《[UCB CS61C][Lab1: C and GDB]》<https://zhuanlan.zhihu.com/p/409918652>
- 知乎《[UCB CS61C][Lab2: Advanced C and Valgrind]》<https://zhuanlan.zhihu.com/p/410399669>
- CS61C Lab/Project 解决过程与笔记 <https://yuruwind.github.io/2025/09/07/CS61C-Lab&Proj-%E8%A7%A3%E5%86%B3%E8%BF%87%E7%A8%8B/>
- CS61C Labs 汇总 <https://theoyu.top/2022/cource/cs61c_labs>
- learncs.site 的 CS61C 实验页（站点为中文社区，内容多为英文原文镜像）<https://www.learncs.site/docs/curriculum-resource/cs61c/labs/lab01>

## 参考来源

- 课程主页（存档）：<https://web.archive.org/web/2021/https://cs61c.org/fa20/>
- 各 lab 页面（存档）：`https://web.archive.org/web/2021/https://cs61c.org/fa20/labs/labXX/`
- 起始代码仓库：<https://github.com/61c-teach/fa20-lab-starter>
- Venus 参考：<https://cs61c.org/resources/venus-reference>
