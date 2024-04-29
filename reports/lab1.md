# Lab 1 实验报告

## 编程作业

题目要求是实现 `sys_task_info()` 系统调用函数，该函数返回当前正在执行的任务的 `TaskInfo`。

`TaskInfo` 中统计的信息有：

- `syscall_times[]`，该任务调用的每个 syscall 的次数；
- `time`，该任务从第一次开始执行到当前所经历的时间。

这两个信息在框架中都没有维护，需要我们自己编写代码维护。

阅读框架代码后发现，任务是存储在 `TaskManager` 中，因此我们考虑在这里面新建变量来维护这两个信息。至于为什么不在 `TaskControlBlock` 中维护，是因为 TCB 维护的是任务的状态，而这两个信息和任务的状态没有太多的关系。

我们可以用一个二维数组存储每个任务的 `syscall_times[]`，一个一维数组来存储每个任务第一次开始执行的时间点，`time` 就等于当前时间减去这个值。除此之外，还需要新建对应的 getter 函数，来根据 `current_task` 返回当前任务的信息。

接下来是何时更新这两个值。

`time` 需要在线程第一次被执行的时候赋值，因此需要在 `run_first_task()` 和 `run_next_task()` 中更新，每次更新需要判断该值是否被赋值过，若没有赋值过才进行赋值，因为我们维护的是第一次执行的时间点。

而 `syscall_times[]` 则是在 `syscall()` 中直接更新就好了。为了代码的安全性，我们最好在 `TaskManager` 中新建一个函数，根据 `syscall_id` 增加对应的值，而不是直接把 `syscall_times` 设成 `pub`。

最后，只需要在 `sys_task_info` 中调用对应的函数，创建新的 `TaskInfo` 返回就好了。

## 简答题

### 第一题

如下图所示，从上到下依次是 `ch2b_bad_address.rs`、`ch2b_bad_instructions.rs` 和 `ch2b_bad_register.rs` 的输出内容。最后一个是 `ch2b_hello_world.rs`，能够正常运行。

![error](2024-04-27-22-58-16.png)

可以看到，第一个程序触发了 `StoreFault`，因为其试图访问一个非法的内存地址。后面两个触发了 `IllegalInstruction`，代表处理器尝试执行一个不存在的指令，这是因为当前处理器运行在 U 态，而程序想要执行 S 态指令。

我使用的 RUST-SBI 版本是 0.2.0-alpha.2。

### 第二题

