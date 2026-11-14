# Lab 2：进阶 C（Makefile、位运算、动态内存）（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 2（Wayback 存档）整理翻译。英文原文见 `original-en/lab02.md`。
> 截止日期：EOL（End-Of-Labs）9 月 11 日（周五）。

## 学习目标

- 通过位运算的组合完成特定的位操作。
- 识别动态内存管理中的潜在问题（泄漏、悬垂指针、未初始化等）。

## 准备

```bash
git pull starter master
# 若报 starter 不是 git 仓库：
git remote add starter https://github.com/61c-teach/fa20-lab-starter.git
git pull starter master
```

## 练习 0：Makefile

Lab 1 里手动敲一长串 gcc 命令很繁琐；文件一多就不可维护。绝大多数 C 程序员用 **Makefile**：一个文本文件，包含若干"规则（rule）"，每条规则给出编译一个 target 的命令。敲 `make <target>` 即可构建。

阅读本 lab 附带的 `Makefile`，在 `make.txt` 中回答（可上网查）：

1. 哪个 target 属于"删除所有已编译程序"的规则？
2. 哪个 target 属于"构建全部程序"的规则？
3. 当前使用的是哪个编译器？
4. 当前使用哪个 C 标准？
5. 在 makefile 中如何引用变量 `FOO`？
6. "Darwin" 指哪个操作系统？
7. 哪一行用目标文件生成 `lfsr` 程序？（写出该行行号。）

## 练习 1：位运算

在 `bit_ops.c` 中实现三个函数：

```c
// 返回 x 的第 n 位，假设 0 <= n <= 31
unsigned get_bit(unsigned x, unsigned n);

// 把 x 的第 n 位设为 v，假设 0 <= n <= 31，v 为 0 或 1
void set_bit(unsigned *x, unsigned n, unsigned v);

// 翻转 x 的第 n 位，假设 0 <= n <= 31
void flip_bit(unsigned *x, unsigned n);
```

**限制**：只允许使用位运算 `& | ^ ~ << >>`；**不许**用 for/while 循环、条件语句、取模 `%`、除法、加法、减法、乘法。

参考实现：

```c
unsigned get_bit(unsigned x, unsigned n) { return (x >> n) & 1u; }

void set_bit(unsigned *x, unsigned n, unsigned v) {
    *x = (*x & ~(1u << n)) | (v << n);
}

void flip_bit(unsigned *x, unsigned n) { *x ^= (1u << n); }
```

思路：先用 `& ~(1u << n)` 清位，再 `| (v << n)` 置位；翻转就是与 `1u << n` 异或。

测试：

```bash
make bit_ops
./bit_ops
```

## 练习 2：线性反馈移位寄存器（LFSR）

LFSR 用于数字电视、CDMA 手机、以太网、USB 3.0 等。在 `lfsr.c` 中实现 `lfsr_calculate()`：

- 每次调用把寄存器内容**右移 1 位**；
- 这不是逻辑右移也不是算术右移：左侧移入 1 位，其值等于原第 **0、2、3、5** 位的异或（XOR）；
- 正确实现后，在回到起始数之前应输出全部 65535 个正 16 位整数；
- 最左为 MSB，最右为 LSB。

参考：

```c
unsigned lsb  = reg & 1u;
unsigned tap  = ((reg >> 0) ^ (reg >> 2) ^ (reg >> 3) ^ (reg >> 5)) & 1u;
reg = (reg >> 1) | (tap << 15);
```

测试：

```bash
make lfsr
./lfsr
```

期望输出的前几项：1, 5185, 38801, 52819, 21116, 54726, …，最后打印

```
Got 65535 numbers before cycling!
Congratulations! It works!
```

## 练习 3：内存管理（实现变长数组 vector）

文件：`vector.h`、`vector.c`、`vector-test.c`。要求：

1. 解释 `bad_vector_new()` 与 `also_bad_vector_new()` 为什么不好；
2. 补全 `vector_new()`、`vector_get()`、`vector_delete()`、`vector_set()`（以及 `vector.h` 中的函数声明），使 `vector-test.c` 无内存错误；
3. 在 Makefile 中为 `vector-test` 目标补一条规则。

函数行为见代码注释；已实现的函数展示了数据结构用法。**约定：凡用户未设置的元素都视为 0**——注意 `malloc()` **不会**清零内存。

两个坏函数的思路提示（考虑内存使用）：

- `bad_vector_new()` 返回**栈上局部变量**的地址，函数返回后该内存失效 → 悬垂指针。
- `also_bad_vector_new()` 按值返回 struct：表面能跑，但内部 `data` 指针指向堆，谁负责 `free` 不清晰，容易泄漏或重复释放。

正确做法：`vector_t` 本身也 `malloc` 在堆上，返回其指针。

参考实现（注意扩容、清零、释放顺序）：

```c
vector_t *vector_new() {
    vector_t *v = malloc(sizeof(vector_t));
    if (!v) allocation_failed();
    v->size = 16;
    v->data = malloc(sizeof(int) * v->size);
    if (!v->data) { free(v); allocation_failed(); }
    memset(v->data, 0, sizeof(int) * v->size);
    return v;
}

void vector_delete(vector_t *v) {
    if (!v) return;
    free(v->data);   // 先释放数据
    free(v);         // 再释放结构体
}
```

`vector_set` 在 `loc >= size` 时需要扩容（`realloc`，建议翻倍），并把新增部分清零；`vector_get` 需处理 `NULL` vector 与越界。

### 正确性与内存检查

```bash
# 1) 功能正确性
make vector-test
./vector-test

# 2) Valgrind 内存检查
make vector-memcheck
```

`vector-memcheck` 实际执行：

```bash
valgrind --tool=memcheck --leak-check=full --track-origins=yes [系统相关参数] ./<executable>
```

自己解释每个 flag 的含义（可回看 Lab 1 练习 4）。看输出的最后一行：

- 有 bug：`ERROR SUMMARY: 1200039 errors from 24 contexts (suppressed: 18 from 18)`
- 干净：`ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 18 from 18)`

`suppressed` 的数量可以忽略。也可以配合 CGDB 或在代码里加 `printf` 调试。

## Checkoff

提交到 Gradescope 的 **Lab Autograder**。

练习 3 的 checkoff 问题：

- 向 TA/AI 解释 `bad_vector_new()` 与 `also_bad_vector_new()` 为什么不好；
- 展示 `make vector-memcheck` 的输出。
