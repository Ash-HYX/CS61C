# Lab 11：Spark 与 MapReduce（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 11（Wayback 存档）整理翻译。英文原文见 `original-en/lab11.md`。
> 截止日期：12 月 4 日（周五）实验课结束。

## 准备

```bash
$ git pull starter master
```

> **注意：本 lab 只能在 Hive 机器上运行。**

本 lab 用 Python 写 Spark，需要先建虚拟环境（使用正确版本的 Python）：

```bash
$ conda create --name lab11env python=2.7
# 提示安装包时输入 y（约 30 秒），可忽略 conda 过旧的警告
$ source activate lab11env
```

> 退出虚拟环境后若继续做 lab，必须重新执行 `source activate lab11env`，否则 Spark 无法运行。

另外在 Hive 上要设置 `JAVA_HOME`：

```bash
$ export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

> 不设置的话，Spark 会报找不到正确 Java 版本的错误。

## 背景

课堂上讲了集群计算（特别是 MapReduce）的原理，现在动手实践。本 lab 使用 **Spark**——它正是在 Berkeley 开发、2013 年捐给 Apache 基金会的。我们用 Python 编写 Map/Reduce 例程。

Spark 官网有安装说明，可以尝试装到本地，但 ssh 到实验机更省事。

### 避免全局变量

Spark 中**不要用全局变量**！这违背多任务并行的初衷，多个任务访问同一全局变量会形成瓶颈。因此大多数算法都不使用全局变量实现。

### 文档与资源

- Spark 快速编程指南（点 Python 标签）：https://spark.apache.org/docs/latest/rdd-programming-guide.html
- 本 lab 使用 Spark 1.1.0，Python API 文档：https://spark.apache.org/docs/latest/api/python/index.html （文档版本号可能不同，API 应兼容）。

## 练习

> 不同练习可能需要重新考虑 `map()`、`flat_map()`、`reduce()` 的实现与调用顺序。

示例输入在 `seqFiles/`：

1. `billOfRights.txt.seq` —— 美国宪法前 10 条修正案，按文档拆分（很小）
2. `complete-works-mark-twain.txt.seq` —— 马克·吐温全集（中等大小）

`.seq` 是 Spark 可读的序列文件，**不是人类可读的**。Spark 也支持其他输入格式，本 lab 不需要。对应的可读文本在 `textFiles/`，可用于了解内容。

建议先在 `billOfRights` 上验证正确性、调试代码。

> 提醒：只能在 Hive 上运行，且必须先激活 Python 虚拟环境。

### 练习 0：为 Spark 生成输入文件

**Step 1**：去 [Project Gutenberg](https://www.gutenberg.org/) 选一部作品，下载 "Plain Text UTF-8" 版本到 lab 目录。

**Step 2**：用文本编辑器打开，在希望 Spark 切分文档的位置各插入单独一行 `—END.OF.DOCUMENT—`。导入器会给每段分配任意 key（如 `doc_xyz`），value 是两个标记之间的内容。按章节/段落切成大小合理的块即可，不必花太多时间。

**Step 3**：用导入器生成 `.seq` 文件（导入器本身是用 Hadoop 写的 MapReduce 程序，见 `Importer.java`，细节不重要）：

```bash
$ make generate-input myinput=YOUR_FILE_FROM_STEP_2.txt
```

生成的 `.seq` 在 `seqFiles/`。做完其他练习后，也在你自己的文件上跑一跑。

### 练习 1：运行 Word Count

`wordCount.py` 已写好，先读懂它做什么。

```bash
$ spark-submit wordCount.py seqFiles/billOfRights.txt.seq
```

`spark-submit` 接收描述 map/reduce 步骤的 Python 文件，把它分发到各 worker（本 lab 上是本机多个处理器），并以 `.seq` 作为输入。输出在 `spark-wc-out-wordCount/part-00000`。

再在较大文件上运行：

```bash
$ spark-submit wordCount.py seqFiles/complete-works-mark-twain.txt.seq
```

输出会覆盖同一文件。可在输出中搜 `'the'` 感受结果。

### 练习 2：每个词出现在多少个文档中？

前面用 `—END.OF.DOCUMENT—` 把文本切成多个文档。例如 `billOfRights.txt` 切成 10 个文档（每条修正案一个）。本练习统计**每个词出现在多少个文档中**（不是出现次数）。例如 `"Amendment"` 应出现在全部 10 个文档中。

打开 `perWordDocumentCount.py`（目前与 `wordCount.py` 功能相同），修改为：

- 统计包含每个词的**文档数**；
- 输出按字母序排序。

提示：

- 看 Spark 的 [transformations](https://spark.apache.org/docs/latest/rdd-programming-guide.html#transformations) 列表，找排序、去重相关方法。
- 为区分"词是否出现在某文档"，需要利用**文档 ID**（见 `flatMapFunc` 注释）。
- 不要求沿用给定框架，可增删 transformation，也可把函数改成更有意义的名字。

测试（结果在 `spark-wc-out-perWordDocumentCount/part-00000`）：

```bash
$ spark-submit perWordDocumentCount.py seqFiles/billOfRights.txt.seq
```

#### Checkoff

- 向 TA 解释你对 `perWordDocumentCount.py` 的修改；
- 展示 `billOfRights` 的输出，特别是 `"Amendment"`、`"the"`、`"arms"` 的值是否合理（可对照 `billOfRights.txt` 验证）。

### 练习 3：全文索引（Full Text Index）

对每个词和它出现过的每个文档，生成该词在文档中**每次出现**的下标列表。下标 = 从文档开头数起的词序号（第一个词为 0）。输出按词字母序排序，格式大致：

```
(word1  document1-id, word# word# ...)
(word1  document2-id, word# word# ...)
...
(word2  document1-id, word# word# ...)
...
```

- 每个词在**每个**出现过的文档各占一行；
- 每对 (词, 文档) 只有**一个**下标列表；
- 需要记录文档 ID。

例：文档 `With great power comes great responsibility` 的输出：

```
('comes doc_somerandomnumbers', 3)
('great doc_somerandomnumbers', 1 4)
('power doc_somerandomnumbers', 2)
('responsibility doc_somerandomnumbers', 5)
('With doc_somerandomnumbers', 0)
```

编辑 `createIndices.py`。不一定要用上提供的所有函数，没用的可以删掉调用；输出同样要排序。

```bash
$ spark-submit createIndices.py seqFiles/billOfRights.txt.seq
```

结果在 `spark-wc-out-createIndices/part-00000`，文件很大，可用 `cat`、`head`、`more`、`grep`：

```bash
$ head -25 OUTPUTFILE       # 前 25 行
$ cat OUTPUTFILE | more     # 逐屏查看（空格翻页）
$ cat OUTPUTFILE | grep the # 只显示含 'the' 的行（区分大小写）
```

验证：打开 `billOfRights.txt`，挑几个词，手动数几个下标，确认都在输出中。

#### Checkoff

- 向 TA 解释 `createIndices.py`；
- 运行：

  ```bash
  $ spark-submit createIndices.py seqFiles/complete-works-mark-twain.txt.seq
  $ cat spark-wc-out-createIndices/part-00000 | grep Mark | less
  ```

  展示 "Mark" 结果的第一页。

### 练习 4：最流行的词

用 Spark 找出 Bill of Rights 中最流行的词：生成按降序排列的 `(count, word)` 列表。`mostPopular.py` 已从 `wordCount.py` 复制过来作为起点。

提示：`reduceByKey` 之后仍可继续 map；看看 `sortByKey` 的参数能省很多翻文档的时间。

```bash
$ spark-submit mostPopular.py seqFiles/billOfRights.txt.seq
```

结果在 `spark-wc-out-mostPopular/part-00000`。也可以在自己练习 0 下载的书上试试。

#### Checkoff

在 `lab11` 目录运行 `./submit`，然后把代码提交到 Gradescope 的 **Lab Autograder**。**记得提交 `outputs` 文件夹！**

- 练习 2：解释 `perWordDocumentCount.py` 的修改；
- 练习 3：解释 `createIndices.py` 的修改；
- 练习 4：向 TA 解释代码。
