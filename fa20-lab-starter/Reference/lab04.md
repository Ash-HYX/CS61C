# Lab 4：RISC-V 函数与指针（调试）（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 4（Wayback 存档）整理翻译。英文原文见 `original-en/lab04.md`。
> 截止日期：9 月 25 日（周五）实验课结束。

## 目标

- 练习调试 RISC-V 汇编代码。
- 编写使用指针的 RISC-V 函数。

## Venus 模拟器

同上周，使用 **Venus**（https://venus.cs61c.org，仓库 `tools/venus.jar`）。需要复习时查课程网站的 Venus Guide。

## 练习 1：调试 `megalistmanips.s`

Lab 3 你实现了对单链表每个元素应用函数的 `map`；这次数据结构是"**由 int 数组组成的链表**"。C 中数组放在 struct 里时必须显式保存数组长度：

```c
struct node {
    int *arr;
    int size;
    struct node *next;
};

void map(struct node *head, int (*f)(int)) {
    if (!head) return;
    for (int i = 0; i < head->size; i++) {
        head->arr[i] = f(head->arr[i]);
    }
    map(head->next, f);
}
```

### 任务

1. 在 `megalistmanips.s` 的 `map` 函数中找出**五处错误**。阅读 `map` 中（`jr ra` 返回前）所有注释行，确保每行代码确实做了注释所说的事。提示：
   - 调用 `jal` 之前为什么要把东西存到栈上？
   - `add t0, s0, x0` 与 `lw t0, 0(s0)` 有什么区别？
   - 注意 `struct node` 各属性的类型。
   - 只需关注 `map`、`mapLoop`、`done` 三个函数，但建议理解整个程序。
2. **本练习禁止使用额外的 save 寄存器**：调用 `f` 后仍要用的值不能放进 `s2`~`s11`，只能用临时寄存器并遵守 caller/callee 约定。给定实现只使用 `s0`、`s1`。
3. 把五处错误按顺序列成清单，并写出你的修改。
4. 修改后的代码保存在 `megalistmanips.s`，并做调用约定检查：

   ```bash
   java -jar venus.jar megalistmanips.s -cc
   ```

   CC checker 应报告 0 条警告。

   > 注意：CC checker **不会**检查你是否用了 `s2`~`s11`，但 autograder 会，所以必须遵守第 2 条要求。

5. 在网页界面运行 `megalistmanips`，参考输出：

   ```
   Lists before:
   5 2 7 8 1
   1 6 3 8 4
   5 2 7 4 3
   1 2 3 4 7
   5 6 7 8 9

   Lists after:
   30 6 56 72 2
   2 42 12 72 20
   30 6 56 20 12
   2 6 12 20 56
   30 42 56 72 90
   ```

### 阶段自检（不计入 checkoff）

- 会在 Venus 中调试：单步、查看寄存器。
- 理解 RISC-V 如何访问内存。
- 理解 RISC-V 的 CALLER/CALLEE 约定。

## 练习 2：不用分支实现函数

离散函数 `f` 定义在 `{-3,-2,-1,0,1,2,3}` 上：

```
f(-3) = 6
f(-2) = 61
f(-1) = 17
f(0)  = -38
f(1)  = 19
f(2)  = 42
f(3)  = 5
```

### 任务

1. 在 `discrete_fn.s` 中用 RISC-V 实现该函数，**不得使用任何 branch / jump 指令**。
2. 结果保存在 `discrete_fn.s`。

提示：如何从"动态地址"加载一个字（`lw`）？把输入值映射为数组下标，查表即可。

## Checkoff

提交到 Gradescope 的 **Lab Autograder**（与上周相同）。
