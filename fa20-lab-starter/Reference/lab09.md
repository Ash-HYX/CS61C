# Lab 9：SIMD 指令与循环展开（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 9（Wayback 存档）整理翻译。英文原文见 `original-en/lab09.md`。
> 截止日期：EOD 11 月 13 日（周五）。

## 目标

- 学习并使用各种 SIMD 函数实现数据级并行。
- 把特定函数 SIMD 化。
- 了解循环展开（loop unrolling）及其原理。

## 准备

```bash
git pull starter master
```

> **免责声明**：所有使用 SSE 指令的代码保证能在 **Hive 机器**上运行，其他地方可能不行。较新的处理器大多支持 SSE intrinsics，但可能看不到准确的加速比。理想情况下 ssh 到 Hive 上做本 lab；后面问到的性能特征也更容易在 Hive 上体现。

## 练习 1：熟悉 SIMD 函数

面对大量 SIMD intrinsic，需要学会查文档。打开 [Intel Intrinsics Guide](https://software.intel.com/sites/landingpage/IntrinsicsGuide/)，勾选所有以 "SSE" 开头的项，找出下列操作对应的 128 位 intrinsic：

- 4 个单精度浮点数（float）的除法
- 16 个有符号 8 位整数（char）的 max 操作
- 8 个有符号 16 位整数（short）的算术右移

提示：名字带 "epi"/"pi" 的是整数；带 "**ps**"/"**pd**" 的分别是单精度/双精度浮点。

可视化工具：

- [SIMD Visualiser](https://piotte13.github.io/SIMD-Visualiser/)：输入代码看向量与函数如何协同工作。
- [Compiler Explorer](https://godbolt.org/)：看编译器生成的汇编，对优化很有帮助。

SIMD 使用建议：

- 注意内存对齐。例如 `_mm256_load_pd` 不能用于未对齐数据，需要用 `_mm256_loadu_pd`。尽量让数据对齐（可用特殊的内存分配 API）。对齐时 aligned load/store 性能与 aligned store 相同；对齐 load 还能折叠成内存操作数，减小代码体积。现代 CPU 对未对齐 load 支持很好，但跨 cache line 边界时仍有明显性能损失。
- 回忆流水线冒险：数据冒险会严重拖慢性能，若性能不达预期，检查相邻 SIMD 操作的数据依赖。

## 练习 2：编写 SIMD 代码

### 常见错误（加粗的是**不要**做的）

- **把求和向量存进 `long long int` 数组**。要用 `int` 数组。函数返回值确实是 `long long int`，因为 `int` 装不下外层循环所有迭代的总和；但单次外层迭代的和 `int` 足够。所以每次外层迭代结束把和向量存进 `int` 数组，再把总和累加到最终结果 `result`。
- **重复初始化求和向量**。累加时**不要**重新声明一个新的 sum 向量。
- **忘记尾处理（tail）的条件判断**。加入 sum 之前我们一直在检查什么条件？
- **往未初始化的数组里累加**。未初始化就累加等于加垃圾。先 `storeu` 再累加是可以的。

文件：`simd.c`（含对超大数组求和的代码，会随机重复 `1 << 16` 次，不用管；`clock()` 用于计时），`test_simd.c`（含 `main`）。

### 任务

把 `simd.c` 中朴素的 `sum()` 内层循环 SIMD 化，实现 `sum_simd()`。需要用到以下 intrinsic：

| Intrinsic | 作用 |
|---|---|
| `__m128i _mm_setzero_si128()` | 返回 128 位全零向量 |
| `__m128i _mm_loadu_si128(__m128i *p)` | 加载指针 p 处的 128 位向量 |
| `__m128i _mm_add_epi32(__m128i a, __m128i b)` | 逐元素 32 位整数加法 |
| `void _mm_storeu_si128(__m128i *p, __m128i a)` | 把向量 a 存到 p |
| `__m128i _mm_cmpgt_epi32(__m128i a, __m128i b)` | 逐元素比较 `a_i > b_i`，真为 32 位全 1，假为全 0 |
| `__m128i _mm_and_si128(__m128i a, __m128i b)` | 逐元素按位与 |

思路：SSE intrinsic 是对向量中多个数据并行操作的函数，比循环逐元素快。求和结构是"遍历数组、每次把元素加到 running sum"；向量化就是一次把若干元素加到 sum 向量，最后再把向量各分量汇总成标量和。

- 提示 1：`__m128i` 是 128 位向量类型，用来装 4 个 32 位 int。
- 提示 2：代码已提供含 4 个 127 的向量 `_127`，用于内层循环条件比较。
- 提示 3：内层循环结束前**不要**调用 store（`_mm_storeu_si128`）——store 开销大，每次迭代都 store 反而变慢；但也不能等到外层循环全部结束才 store，否则可能溢出。
- 提示 4：不要把 `__m128i` 当数组下标访问；先用 `storeu` 存到数组，再按元素索引。
- 提示 5：仔细看函数声明——loadu/storeu 参数是 `__m128i*`，可以把 int 数组强制转换；不转换会有一堆编译警告。

```bash
make simd
./simd
```

自检：Hive 上朴素版约 7 秒，SIMD 版应约 1~2 秒。

## 练习 3：循环展开

**概念**：循环展开（loop unrolling）——每次迭代做更多操作，减少循环次数与循环控制开销（想想为什么会有额外周期）。理论上把循环体复制 n 遍会更快，但代码不美观。

简单例子（数组前 n 项求和）：

```c
int total = 0;
for (int i = 0; i < n; i++) {
    total += arr[i];
}
```

展开 4 次，并为 `n` 不是 4 的倍数的情况保留尾处理：

```c
int total = 0;
for (int i = 0; i < n / 4 * 4; i += 4) {
    total += arr[i];
    total += arr[i + 1];
    total += arr[i + 2];
    total += arr[i + 3];
}
for (i = n / 4 * 4; i < n; i++) {
    total += arr[i];
}
```

汇编层面，展开后每次循环做 4 次 load+add，循环计数一次加 4，分支次数减少，因此更快。可参考提供的 `sum_unrolled()`。

### 任务

在 `simd.c` 中，把 `sum_simd()` 的代码复制到 `sum_simd_unrolled()`，并**展开 4 次**，别忘了尾处理。

```bash
make simd
./simd
```

应比 `sum_simd` 再快一点（零点几秒量级）。

## Checkoff

提交到 Gradescope 的 **Lab Autograder**。

check-in 时解释 `sum_simd()` 与 `sum_simd_unrolled()` 的实现：SIMD 比朴素版快多少？展开又带来多少提升（以及为什么能提升）？
