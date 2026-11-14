# Lab 0：课程介绍与环境搭建（中文说明）

> 依据 CS61C Fall 2020 官方 Lab 0 页面（cs61c.org/fa20/labs/lab00，已通过 Wayback Machine 存档）整理翻译。英文原文见 `original-en/lab00.md`。
> 截止日期：9 月 11 日（周五）——原定 9 月 4 日，lab00 单独延长一周。

欢迎来到 CS61C！这份实验有点长，但请仔细读完：它覆盖了后面整个学期都要用的工具和流程。

## 学习目标

- 能描述并遵守所有课程政策，包括 lab checkoff（当面验收）政策。
- 注册 GitHub、Gradescope、Hive 机器、Piazza 等课程服务。
- 熟悉 Linux 与 Git 命令。

## 术语

- **TA/GSI/uGSI**：助教。
- **AI**：Academic Intern（学业实习生），课程团队成员，负责 checkoff、OH、Piazza。本课程中 AI 不是"人工智能"。
- **OH**：Office Hours，答疑时间。
- **Hive**：EECS 系维护的一组教学服务器（`hive1`~`hive30`），文件通过共享网络存储，在所有机器上一致。

## 练习 0：自我介绍

本学期约 1200 人注册（+50 候补）。请尽快（8 月 28 日周五结束前）填写课程介绍表单，内容不影响成绩。

## 练习 1：课程政策

阅读课程网站 policies 页面，重点关注 **Grading（评分）**、**Labs（实验）**、**Academic Dishonesty and Cheating（学术不端）** 三节。
然后完成 Gradescope 上的 "Lab 0: Course Policies Quiz"。

## 练习 2：环境搭建

### 需要注册的服务

| 服务 | 用途 |
|---|---|
| Piazza | 课程论坛，所有通知和提问 |
| Hive（EECS 教学账号） | SSH 登录教学服务器，跑代码、测试 |
| Gradescope | 提交作业、评分、部分测验 |
| GitHub / GitHub Classroom | 分发起始代码、管理你的 lab 仓库 |
| YouTube / Zoom / OH Queue | 讲座录像、线上面谈、答疑排队 |

- 能用 CalNet ID 登录就用 CalNet；需要 Google 登录时用 `@berkeley.edu` 邮箱（Berkeley Google 账号）。
- Gradescope 必须把 `@berkeley.edu` 设为主邮箱，否则可能找不到你的账号。
- 校园外访问受限时可用 Berkeley VPN；校内 `CalVisitor` Wi-Fi 会封 SSH，换 `AirBears2`/`eduroam`。
- 非标准注册（concurrent enrollment、CS47C 等）需要另外填表获取资源权限。

### 教学账号与 Hive

在 WebAccount 注册 EECS 教学账号（形如 `cs61c-???`），用于登录 Hive，也是 checkoff 的主 ID。
如果暂时没有账号，也没关系：填非标准注册表，先继续做实验。

### `ssh`

```bash
$ ssh cs61c-???@hive#.cs.berkeley.edu
```

`hive#` 换成 `hive1`~`hive30` 中任意一台（名单见 Hivemind）。常见故障：

- 长时间无响应：网络/防火墙问题，换网络。
- `Connection refused` / 超时：该机器宕机，换一台。
- `Reserved for cs61c staff`：换一台。
- 提示符样式异常：运行 `/home/ff/cs61c/bin/fix-dotfiles`。

退出：`exit`。修改教学账号密码可 SSH 到 `update.cs.berkeley.edu`。

### `scp`

```bash
$ scp <source> <destination>
$ scp cs61c-???@hive3.cs.berkeley.edu:~/some_folder/example.txt ~/Downloads/
$ scp -r cs61c-???@hive3.cs.berkeley.edu:~/some_folder ~/Downloads/
```

远程路径写作 `username@host:path`。复制目录要加 `-r`。
**注意**：`scp` 要在本地终端运行，在 Hive 上运行等于把文件复制到同一台机器。

### Vim 基础

常用命令：`<Esc>:q` 退出、`:wq` 保存退出、`:w` 保存、`:q!` 强制退出、`i` 插入模式、`/cats` 搜索（`n`/`N` 跳转）、`:set nu` 显示行号。

### 本练习行动项

1. SSH 登录任意 Hive 机器；若提示填写姓名、学号、邮箱（用 Berkeley 邮箱）、代号，就照填。
2. 运行 `check-register` 检查信息；有错则 `re-register`。显示的第一个邮箱必须与 Gradescope 的主/次邮箱一致。

## 练习 3：命令行基础

- 快捷键：`Tab` 补全、上下方向键翻历史、`Ctrl+A` 行首、`Ctrl+E` 行尾、`Ctrl+R` 搜索历史。
- `touch file`：建空文件；`echo "..." > file`：写入文件（覆盖）；`cat`/`less`：查看文件。
- `man command | less`：查看手册；`man -k keyword | less`：按关键词搜索命令。
- `gcc --help`、`echo -e "Hello\nworld"`：flag 通常以 `-` 或 `--` 开头。

## 练习 4：玩转 Git

### 获取 lab 仓库

登录 GitHub，填写 Lab Repository Registration Form，接受 GitHub Classroom 邀请（没有教学账号时表单里填 `zzz`，以后再改）。

### 配置与克隆

```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
$ git clone GITHUB_REPOSITORY_URL labs        # 有教学账号，在 Hive 上
$ git clone GITHUB_REPOSITORY_URL labs_hive   # 无教学账号
```

`git clone` 会自动创建名为 `origin` 的远程，指向你的 GitHub 仓库，本地在 `master` 分支上跟踪 `origin/master`。
再添加课程起始代码远程，以便老师更新：

```bash
$ git remote add starter https://github.com/61c-teach/fa20-lab-starter.git
$ git pull starter master
```

### 找命令（核心练习）

文件在仓库的 `lab00/` 目录。目标是熟悉 manpage 与 Git，不要求一次答对：卡住就写最接近的猜测，autograder 会给提示。

1. 打开 `first_set.txt`，用 manpage 找出对应命令（**记得带上必要 flag**），逐行写进 `answers.txt`：

   ```
   1. first_set_command_1
   2. first_set_command_2
   3. first_set_command_3
   4. first_set_command_4
   ```

   然后 `git add` / `git commit -m "Answered first set"` / `git push`。

2. 把仓库复制回本地：有教学账号则 `exit` 后用 `scp -r` 把整个 `labs` 目录拷回本地；没有则把 `labs_hive` 复制成 `labs`。

3. 在本地 `answers.txt` 的**旧答案上方**插入第二组描述对应的命令（含 flag）：

   - 把文件从一个目录移动到另一个目录
   - 修改文件权限
   - 显示当前所在目录
   - 检查某台主机是否在线

   保存，但**不要** commit / push。

4. 回到 Hive 上的仓库，在旧答案上方插入："显示目录下所有文件的权限"。然后 commit & push，信息 `"Answered third set"`。

5. 回到本地仓库，commit & push，信息 `"Answered second set"`。**此时 Git 会报错**——思考报错含义并解决（不要用 `rebase` 及其 flag）。

   解决后本地 `answers.txt` 应包含全部 9 条描述对应的答案顺序：显示权限、移动文件、改权限、显示当前目录、检查主机在线、按关键词遍历目录树查找、显示进程信息、终止进程、比较两个文件差异。最后 commit & push。

## 提交与 Checkoff

### 提交

把工作 push 到 GitHub lab 仓库；若该 lab 有 autograder，再到 Gradescope 的 `Lab Autograder` 作业提交仓库，确认通过测试。

### Checkoff 规则

- 必须**两人一组**做 lab，checkoff 时两人都**必须**到场。
- 预约：OH Queue → Appointments，周日晚约 9PM (PDT) 放出当周时段，13 分钟一档；两人必须约同一档，每人每 lab 只能约一次。
- 到场：OH Queue 首页 `Upcoming Appointments`，TA 准备好后点绿色 `Join Call` 进入 Zoom。

### 参加 checkoff 前自检

1. 完成每个练习的行动项；有问题类练习要准备好口头回答；改代码的要 push。
2. 有 autograder 的已提交并通过，页面保持打开。
3. 你和搭档都能出席所选时段。
4. 知道自己的邮箱和教学账号用户名。

### 拿到学分

checkoff 结束后，务必在 Gradescope 查看 `Lab## Checkoff` 是否为 `Submitted`；课程组不会事后补录。若显示 `Submitted ???? late` 但确实在正确时段完成，可暂时忽略。

## 检查清单

- [ ] 完成练习 0 的自我介绍表单
- [ ] 完成练习 1 的课程政策测验
- [ ] 注册练习 2 的所有服务
- [ ] 在 Hive 上注册并核对信息（练习 3）
- [ ] 完成练习 4（答案与描述顺序一致）
- [ ] 通过本 lab 的 `Lab Autograder`

## 附录：编辑器与快捷 SSH

- 课程不指定 IDE；至少要会一个终端编辑器（Vim/Emacs/Nano）。也可本地用 VSCode/Sublime 编辑，再用 Git 同步到 Hive。
- 嫌每次输完整 SSH 命令麻烦，可参考 Piazza 帖子 "Useful additional setup information for labs and projects" 配置 SSH 别名/免密。
