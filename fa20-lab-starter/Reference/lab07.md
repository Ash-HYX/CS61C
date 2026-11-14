# Lab 7：Cache（缓存）（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 7（Wayback 存档）整理翻译。英文原文见 `original-en/lab07.md`。
> 截止日期：EOD（当天结束）10 月 30 日（周五）。

## 目标

- 分析内存访问模式如何决定 cache 命中率。
- 找出能产生**高**命中率的访问模式。
- 分析 cache 命中率，并通过优化代码访问顺序获得高命中率。

## 准备

```bash
git pull starter master
```

答案写在 `exercise1.txt`、`exercise2.txt`、`exercise3.txt` 中：

- 数值答案写数字，不要写英文单词（写 `7` 而不是 `seven`）。
- `Checkoff Question` 行会被 autograder 忽略，是给你自己看的。
- autograder 假设原有格式不变：**不要增删行、调换行或改动格式**。

## 在 Venus 中可视化 Cache

`cache.s` 是本次的工具。先通读文件，尤其顶部标有 "PSEUDOCODE" 的部分：运行后，程序要么把数组某些元素清零（option 0），要么自增（option 1）。

- 访问**哪些**元素由 **Step Size**（`a1`）决定；
- 访问**多少次**由 **Rep Count**（`a2`）决定；
- **Option**（`a3`）也会影响行为；cache 参数本身同样影响结果。

每个场景的操作步骤：

1. 把 `cache.s` 内容粘贴到 [Venus](https://venus.cs61c.org/)；
2. 按场景要求修改 `main` 中被注释的 `li` 指令立即数，设置 Program Parameters；
3. `Simulator -> Cache`；
4. 按场景设置 Cache Parameters；
5. 执行代码。只有**数据**访存（load/store）会显示；取指不显示（指令在单独的指令 cache 中）。

重置代码会同时重置 cache 命中率。**重要**：一次跑完只能看到最终状态；建议在 `wordLoop` 循环中、每次访存前后设**断点**，观察每次命中/缺失的来源。

## 复习：命中/缺失策略

Venus 模拟的是 **write-through + write-allocate** cache。

写命中策略：

- **Write-back**：只写 cache，置 dirty 位；驱逐时若 dirty 则写回内存。写延迟小，但硬件复杂。
- **Write-through**：同时写 cache 和内存。写延迟较大，但硬件简单，内存始终是最新数据。
- **Write-around**：只写内存；若块在 cache 中则置为 invalid。本质上没有写命中。

写缺失策略：

- **Write-allocate**：写缺失时把块调入 cache。
- **No write-allocate**：写缺失时只更新内存，不调入 cache。

替换策略（通常按有用程度排序）：

- **LRU**：替换最久未使用的块。
- **Random**：随机替换。
- **MRU**：替换最近使用的块。

要点：**在 write-through cache 中，虽然写操作也更新内存，但因为同时写了 cache，所以对已在 cache 中的块写入仍算 write hit。**

常见疑问：通常 write-back 配 write-allocate、write-through 配 no-write-allocate。但对 cache 而言两者在命中时行为相似（都写 cache），所以 Venus 的命中/缺失模式与 write-back 模拟结果相同。

## 练习 1：几个内存访问场景

对每个场景：先**推理**命中率，再模拟验证，并搞清楚**为什么**。

自问（非 checkoff 题）：

- cache block 多大？
- 考虑 step size 后，一个块内能容纳多少次连续访问？
- 整个 cache 能放多少数据？
- 映射到同一 set 的块在内存中相距多远（可能冲突）？
- 相联度是多少？
- 某个块映射到 cache 的哪个位置？
- 判定某次访问命中/缺失时：这块数据之前访问过吗？现在还在 cache 里吗？

### 场景 1

**程序参数**（改代码里的 a 寄存器）：

- Array Size (`a0`)：128（字节）
- Step Size (`a1`)：8
- Rep Count (`a2`)：4
- Option (`a3`)：0

**Cache 参数**（在 Cache 标签设置）：

- Cache Levels：1
- Block Size：8
- Number of Blocks：4
- Enable：绿色（开启）
- Placement Policy：Direct Mapped
- Associativity：1（Venus 不允许改——想想为什么？）
- Block Replacement Policy：LRU

提示：纸上画出 32 位地址的 tag:index:offset 划分，算出各内存地址映射到哪个 set。

#### 任务

1. 是哪两个参数的组合造成了你观察到的命中率？按 "\[参数 A\]，\[参数 B\]" 回答，补全："因为 \[参数 A\] 的字节数恰好等于 \[参数 B\] 的字节数。"（注意 "cache 大小" 也是由 block size 与块数隐含设定的合法参数。）
2. 若 Rep Count 任意增大，命中率是多少？用小数回答（如 100% 写 "1.0"）。（Checkoff Q1：为什么？）
3. 修改**一个**程序参数以提高命中率：按 "\[参数\]，\[值\]" 回答。（不要求访问相同元素，但值必须合法。Checkoff Q2：有两个程序参数可以改，能都列出来吗？）

### 场景 2

**程序参数**：

- Array Size (`a0`)：256（字节）
- Step Size (`a1`)：2
- Rep Count (`a2`)：1
- Option (`a3`)：1

**Cache 参数**：

- Cache Levels：1
- Block Size：16
- Number of Blocks：16
- Enable：绿色
- Placement Policy：N-Way Set Associative
- Associativity：4
- Block Replacement Policy：LRU

#### 任务

1. **内层循环**（不是 Rep Count 那层）每次迭代有多少次**内存访问**？
2. **重复的命中/缺失模式**是什么？按 "MMHHMH" 这样的形式回答，要求是**最短**的重复模式。（Checkoff Q3：为什么？并用该模式解释命中率。）
3. 其他不变、Rep Count 趋于无穷时，命中率趋近多少？改参数跑一下验证，用小数回答。（Checkoff Q4：为什么会这样？）

你应注意到本场景命中率相当高；若不明白，比较数组大小与 cache 大小。

进一步思考：若程序对一个**远大于 cache** 的大数组迭代 Rep Count 次，每次 Rep 对数组元素应用不同函数（如 Rep=1024 就应用 1024 个不同函数，每个 Rep 一个），如何重构数组访问以获得本场景那样的高命中率？假设各元素相互独立（Rep `k` 作用于 `arr[i+1]` 与作用于 `arr[i]` 的先后无所谓）。

**Checkoff Q5**：提示——不要把整个大数组一次遍历完（远大于 cache，会降低时间局部性）。应一次访问数组的**一部分**，把**所有 Rep** 都作用于这一部分，做完再移到下一部分，使这部分在 cache 中保持"热"。填空的三个空是同一个概念（不是某个术语）。

### 场景 3

**程序参数**：

- Array Size (`a0`)：128（字节）
- Step Size (`a1`)：1
- Rep Count (`a2`)：1
- Option (`a3`)：0

**Cache 参数**：Cache Levels = 2。

**L1**（在 replacement policy 旁的下拉框选 L1）：

- Block Size：8
- Number of Blocks：8
- Enable：绿色
- Placement Policy：Direct Mapped
- Associativity：1
- Block Replacement Policy：LRU

**L2**（下拉框选 L2）：

- Block Size：8
- Number of Blocks：16
- Enable：绿色
- Placement Policy：Direct Mapped
- Associativity：1
- Block Replacement Policy：LRU

#### 任务

1. L1 命中率、L2 命中率、总体命中率各是多少？按 "\[L1 HR\]，\[L2 HR\]，\[Overall HR\]" 回答，保留两位小数。
2. 对 L1 总共有多少次访问？其中多少次缺失？按 "\[L1 访问数\]，\[L1 缺失数\]" 回答。
3. 对 L2 总共有多少次访问？提示：这与 L1 有什么关系（L1 什么时候才会去访问 L2）？
4. 修改哪个程序参数能提高 L2 命中率、同时保持 L1 命中率不变？（Checkoff Q6：为什么？）
5. 当 (1) 增加 L1 块数，或 (2) 增大 L1 block size 时，L1 与 L2 命中率分别下降（-）、不变（=）还是上升（+）？按 "\[1_L1\]，\[1_L2\]，\[2_L1\]，\[2_L2\]" 回答（如 "=, -, =, +"）。

## 练习 2：循环顺序与矩阵乘法

矩阵按**列主序**存于一维数组，n×n 矩阵乘法：

```c
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)
        for (int k = 0; k < n; k++)
            C[i+j*n] += A[i+k*n] * B[k+j*n];
```

上面循环顺序为 `i, j, k`。看最内层（`k`）循环：

- 访问 B 的 stride 为 1
- 访问 A 的 stride 为 n
- 访问 C 的 stride 为 0

**计算正确性与循环顺序无关**，但访问顺序**极大影响性能**：空间/时间局部性越好，命中率越高。

查看 `matrixMultiply.c`，其中有 6 种三重循环实现。

**任务**：分析另外五种实现的 stride。

Makefile 用 `-O3` 优化（必须保留）：

```bash
make ex2
```

程序会跑 6 种实现并输出速度，单位 Gflops/s（每秒十亿次浮点运算）——**数字越大越快**。

#### 任务

1. 对 1000×1000 矩阵，哪两种顺序最快？按 "\[顺序1\]，\[顺序2\]" 回答（如 "ijk, ikj"）。（Checkoff Q1：为什么？）
2. 哪两种最慢？（Checkoff Q2：为什么？）

**Checkoff Q3**：最内层循环跨矩阵的 stride 方式如何影响性能？

## 练习 3：Cache Blocking 与矩阵转置

> 建议在 **Hive 机器**上做（历史上 Hive 跑测试快一倍）。在 Soda 27x 机房也可从教学机 ssh 到 Hive。

### 矩阵转置

交换矩阵的行与列，A 的转置记为 Aᵀ。

### Cache Blocking

上面的矩阵乘法为了算 C 的一个值要跨整个 A、B，几乎不复用缓存数据。**Cache blocking** 通过提升时间/空间局部性来降低缺失率：矩阵转置时，一次处理一个子块（block）。

把 A 的每个子矩阵 A(i,j) 逐个转置到输出矩阵的最终位置；逐个完成子块，可把访存集中在这小块内存上，提高局部性，从而更快。

正确实现会有显著性能提升。

### 任务

在 `transpose.c` 的 `transpose_blocking()` 中实现 cache blocking。**不能假设矩阵宽度 `n` 是 blocksize 的整数倍**。该函数默认什么都不做，benchmark 会报错。

```bash
make ex3
./transpose <n> <blocksize>
```

可用 `n=10000`、`blocksize=33` 验证：分块版本应明显更快。

入门提示：

- 先看 `transpose_naive`：外层循环一次迭代中，`y` 会纵跨整个 `src` 才重置为 0；也就是 `x` 要等 `y` 从 0 到 n 走完才更新。我们要改掉这种行为——按块"步进"，而不是扫完一整列/行。
- TL;DR：一次填满 `dst` 的一个 `blocksize × blocksize` 方块。
- 标准解法需要 **4 个 for 循环**。
- 因为 `n` 未必是 blocksize 的倍数，最右一列块和最下一行块会被截断。可先假设整除实现，再在越界时跳过（特判）。

完成后把结果与回答记录到 `exercise3.txt`（该文件不参与 autograder）。

### Part 1：改变数组大小

固定 `blocksize = 20`，分别取 `n = 100, 1000, 2000, 5000, 10000`，把输出记录到 `exercise3.txt`。

- **Checkoff Q1**：从多大开始，分块版本比非分块版本快？
- **Checkoff Q2**：为什么矩阵要足够大，cache blocking 才能超过非分块代码？

（自检：矩阵不够大时，分块版并不更快。）

### Part 2：改变 blocksize

固定 `n = 10000`，分别取 `blocksize = 50, 100, 500, 1000, 5000`，记录输出。

- **Checkoff Q3**：性能随 blocksize 如何变化？为什么？

（自检：随 blocksize 增大，加速比应先朝一个方向变化，再朝另一个方向变化。）

注意：这两个练习我们并不知道机器的 cache 参数，只是让代码表现出更高的局部性，性能就提升了——说明无论 cache 参数如何，局部性好的代码总能受益。

## Checkoff

提交到 Gradescope 的 **Lab Autograder**。

- autograder 忽略空白和大小写，但很简单，无法识别拼写/格式错误。
- 它检查 `exercise1.txt`、`exercise2.txt`、`transpose.c`、`transpose.h`（后者不要改）。

三个练习的 Checkoff Questions 用于和 TA 讨论；建议把回答与性能数据直接写在预留位置，checkoff 时不用现找。本 lab 对理解 cache 很关键，checkoff 时多问问题。
