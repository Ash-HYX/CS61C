# Lab 10：线程级并行（OpenMP）与多进程（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 10（Wayback 存档）整理翻译。英文原文见 `original-en/lab10.md`。
> 截止日期：11 月 20 日（周五）实验课结束。

## 目标

- 了解基本 OpenMP 指令。
- 用代码理解 `#pragma omp for` 的两种可能实现，认识伪共享（false sharing）。
- 了解基本的多进程编程。

## 准备

```bash
git pull starter master
```

## 第一部分：用 OpenMP 做多线程编程

OpenMP = Open specification for Multi-Processing，提供 C 接口。它不是语言的一部分，大多数特性是给编译器的**指令（directive）**。

优点：

- 接口简单，可把程序划分为串行区与并行区。
- 同步控制方便（POSIX 线程的数据竞争 bug 极难排查）。

本 lab 练习基本用法。可以在自己机器上装 OpenMP，但 **Hive 已预装好，最省事**。

### 练习 1：OpenMP Hello World

```c
int main() {
    #pragma omp parallel
    {
        int thread_ID = omp_get_thread_num();
        printf(" hello world %d\n", thread_ID);
    }
}
```

程序会 fork 默认数量的线程，每个线程打印 "hello world" 及自己的线程号。

- 用环境变量 `OMP_NUM_THREADS` 或 `omp_set_num_threads()` 改变线程数。
- `#pragma` 告诉编译器本行剩余部分是指令；`omp` 表示 OpenMP；`parallel` 表示后面的 `{ }` 代码块可并行执行。

```bash
make hello && ./hello
```

多跑几次会发现输出顺序不固定：并行区内 OpenMP 不保证线程间顺序。注意 `thread_ID` 是**各线程私有**的：一般地，并行块内声明的变量对每个线程私有，块外声明的变量是全局共享的。

> 接下来的练习都在 `omp_apps.c` 中修改函数。

### 练习 2：向量加法

向量加法天然并行。`v_add` 返回 x 与 y 逐元素相加的数组。最初的错误写法：

```c
void v_add(double* x, double* y, double* z) {
    #pragma omp parallel
    {
        for (int i = 0; i < ARRAY_SIZE; i++)
            z[i] = x[i] + y[i];
    }
}
```

```bash
make v_add && ./v_add
```

测试框架会改变线程数并计时。你会看到线程越多反而越慢：每个线程都执行了 `parallel` 块内的**全部**代码，8 个线程就把向量加了 8 遍。应把 for 循环拆分给各线程，每个线程只做一部分。

任务：优化 `v_add`（线程数继续增加时加速比可能趋于平缓）。可用：

- `int omp_get_num_threads();`：当前 parallel 块的线程数
- `int omp_get_thread_num();`：当前线程 ID

用**两种不同方法**分别实现：

1. **Slicing（交错）**：每个线程处理"相邻"的加法？——原文定义：线程 0 处理满足 `i % omp_get_num_threads() == 0` 的索引 `i`，线程 1 处理 `i % num_threads == 1` 的，依此类推。（即按线程号交错分配。）
2. **Chunking（分块）**：N 个线程把向量分成 N 个连续块，每个线程只处理自己那一块。

提示：

- 在 for 循环中用上面两个函数决定各线程处理哪些元素。
- `v_add_optimized_chunks` 可能需要特判防止越界，放心写。
- 思考**伪共享（false sharing）**：多个线程写同一 cache line 中不同变量，会导致该 line 在核心间反复失效。

> 本练习要求**手动**拆分工作（这是软件优化中的常见模式）。OpenMP 的 `#pragma omp for` 可以自动拆分独立工作，如下所示——但**本练习禁止使用它**：
>
> ```c
> #pragma omp parallel for
> for (int i = 0; i < ARRAY_SIZE; i++)
>     z[i] = x[i] + y[i];
> ```

### 练习 3：点积

计算两向量点积。看似与 `v_add` 类似，难点在于把各乘积累加到**同一个变量**（reduction）。处理不当会导致**数据竞争（data race）**：多个线程同时读写同一地址。

一种方案是**临界区（critical section）**：同一时刻只允许一个线程执行。OpenMP 提供 `critical` 原语。`omp_apps.c` 中的 `dotp_naive` 就用临界区保护求和：

```bash
make dotp && ./dotp
```

你会看到线程数增加时性能急剧下降：把所有 reduction 工作放进临界区，等于把并行度压成 1，一次只有一个线程做有用功。线程越多争用越激烈。

任务：

- 在 `dotp_manual_optimized` 中**不用** `reduction` 关键字修复性能问题：减少每个线程进入临界区的**次数**（例如每个线程先在私有变量中累加，最后只进一次临界区）。
- 在 `dotp_reduction_optimized` 中用 OpenMP 内置 `reduction` 关键字实现（代码中不应再有 `#pragma omp critical`）。

## 第二部分：多进程编程入门

- **线程**：可由 OS 独立管理的执行序列；同一进程的线程**共享地址空间**。
- **进程**：正在执行的程序实例，拥有自己的地址空间和一到多个线程；是内核提供保护的主要抽象。

区别带来的两点：

1. 线程开销小（内存等资源占用低），通信成本低（直接读写同一地址空间）。
2. 共享内存要小心并发问题：多线程读写同一地址时正确性难以推理。

### 背景：HTTP Web 服务器与多进程

`fork` 系统调用通过复制调用进程创建新进程：

- 成功时，父进程得到子进程 PID，子进程得到 **0**；
- 失败返回负值。

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main() {
    pid_t child_pid;
    printf("Main process id = %d (parent PID = %d)\n",
           (int)getpid(), (int)getppid());
    child_pid = fork();
    if (child_pid != 0)
        printf("Parent: child's process id = %d\n", child_pid);
    else
        printf("Child:  my process id = %d\n", (int)getpid());
    return 0;
}
```

可能输出：

```
Main process id = 9075 (parent PID = 32146)
Parent: child's process id = 9076
Child:  my process id = 9076
```

**注意 `fork()` 会同时返回到父进程和子进程**：父进程得到子进程 PID，子进程得到 0，因此二者在 `if` 处分叉。

要并行的程序是一个基础 HTTP 服务器：

- 创建监听 socket 并绑定端口，等待客户端连接；
- 收到连接后读取并解析 HTTP 请求，返回所请求的文件；
- 本 lab 只响应 "GET"。

串行版已实现：

```bash
make server_basic && ./server_basic
```

- 默认以 `lab10/files/` 为文件目录，监听 8000 端口（可 `./server_basic --port 8080`）。
- 请求方式：浏览器打开 `localhost:8000/文件名`，或命令行 `curl localhost:8000/文件名`（`man curl`）。

需要优化的是 `server_utils.c` 中的 `server_forever`：当前单进程处理完一个请求才接待下一个；若某个请求耗时较长，后面的客户端只能干等。

额外功能：

- 请求 `localhost:8000/report` 会运行 `dotp` 并以文本返回结果；`ARRAY_SIZE` 默认值在 `omp_apps.h`，可用 `./server_basic --dotp-size 10000000` 修改。
- 路由：请求 `/filter/xxx.bmp` 会对图片运行 sobel 边缘检测，返回同时显示原图与处理后图像的网页。`files/` 下有几张示例 bmp；`localhost:8000/girl.bmp` 是原图，`localhost:8000/filter/girl.bmp` 是处理结果。

说明：

- 在 Hive 上通过 ssh 跑服务、用本地浏览器访问：若 ssh 到 `hive9.cs.berkeley.edu` 并以默认配置启动（监听 127.0.0.1:8000），浏览器访问 `http://hive9.cs.berkeley.edu:8000/`；也可用 `curl`。
- 所用 bmp 库很基础，缺少复杂 padding 支持，滤波只对边长为 2 的幂的 bmp 效果好，其他尺寸效果可能很怪。

可选：sobel 已实现，能否用 OpenMP 优化它？也可以自己实现其他图像算法玩。

为模拟耗时计算，处理完请求后会 **sleep 5 秒**。连续发两个请求即可看到第二个请求要等很久。

### 练习

不要用主进程处理请求：**每次 fork 一个子进程处理请求，父进程继续接待新请求**。上面的 `fork` 示例足够完成。

子进程有独立的系统资源（地址空间、寄存器、栈、打开的系统对象句柄、安全上下文等），任务完成后要显式清理退出——参见 `exit` 系统调用。

测试：

```bash
make server_process && ./server_process
```

然后连续请求同一文件，确认第二个请求立即返回。提供了计时脚本 `timer.sh`：一个终端跑服务器（`server_basic` 或 `server_process`），**另一个终端**（必要时 ssh 到 Hive）运行 `./timer.sh`，脚本会报告服务请求所需时间。fork 版本的提升应非常明显。

注意：从命令行 Ctrl+C 杀父进程时，fork 出的子进程不保证被杀死，可能导致默认 8000 端口被占用、无法重启服务器。starter 代码已替你处理（原理超出 61C 范围，CS162 会讲，但鼓励自己看代码）。若端口被僵尸进程占用，可：

```bash
fuser -k 8000/tcp
# 或换端口
./server_process --port 8080
```

（补充：用 fork 响应请求并不是最好的并行方式——现代方案是线程池，开销更低、便于控制服务器负载。）

## 加速比要求

autograder 的测试是本地测试的微调版，除正确性外还检查性能：

- `v_add`：最快版本（adjacent/chunks，取所有线程数中最好）相对 naive 基准 **2x** 加速。
- `dotp`：最快版本（manual/reduction）相对 naive 基准 **9x** 加速。
- 服务器：连续 3 个请求都应在 **100 ms** 内响应（Hive 上；autograder CPU 较弱，要求更宽松）。

## Checkoff

提交代码到 Gradescope 的 **Lab Autograder**。可讨论：

- 练习 2：chunks 与 adjacent 哪个更快？为什么？
- 练习 3：线程数变化时性能差异如何解释？
- 第二部分：fork 方案减少了连续请求的等待时间，但它是否增加了实际计算时间？
