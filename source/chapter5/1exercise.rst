chapter5 练习
====================

- 本节难度： **看似唬人，其实就那样** 

本章任务
--------------------

.. tabs::

    .. group-tab:: Rust

        - 在 os 目录下 ``make run TEST=1`` 加载所有测例， ``test_usertest`` 打包了所有你需要通过的测例，你也可以通过修改这个文件调整本地测试的内容。
        - 你的内核必须前向兼容，能通过前一章的所有测例。即，正确的 merge ch4。
        - 结合文档和代码理解 fork, exec, wait 的逻辑。结合课堂内容回答本章问答问题（注意第二问为选做）。
        - 完成本章编程作业。
        - 最终，完成实验报告并 push 你的 ch5 分支到远程仓库。

    .. group-tab:: C

        - ``make test BASE=1`` 执行 usershell，然后运行 ``ch5b_usertest``。
        - merge ch4 的修改，运行 ``ch5_mergetest`` 检查 merge 是否正确。
        - 结合文档和代码理解 fork, exec, wait 的逻辑。结合课堂内容回答本章问答问题（注意第二问为选做）。
        - 完成本章编程作业。
        - 最终，完成实验报告并 push 你的 ch5 分支到远程仓库。

.. note::

    利用 ``git cherry-pick`` 系列指令，能方便地将前一章分支 commit 移植到本章分支。
  
编程作业
--------------------

进程创建
++++++++++++++++++++

大家一定好奇过为啥进程创建要用 fork + execve 这么一个奇怪的系统调用，就不能直接搞一个新进程吗？思而不学则殆，我们就来试一试！这章的编程练习请大家实现一个完全 DIY 的系统调用 spawn，用以创建一个新进程。

spawn 系统调用定义( `标准spawn看这里 <https://man7.org/linux/man-pages/man3/posix_spawn.3.html>`_ )：

- syscall ID: 400
- C 接口：``int spawn(char *filename)`` 
- Rust 接口：``fn sys_spawn(path: *const u8) -> isize``
- 功能：相当于 fork + exec，新建子进程并执行目标程序。 
- 说明：成功返回子进程id，否则返回 -1。  
- 可能的错误： 
    - 无效的文件名。
    - 进程池满/内存不足等资源错误。  

实现完成之后，你应该能通过 spawn* 对应的所有测例，在 shell 中执行 usertest 来执行所有测试。

.. hint::

    - 注意 fork 的执行流，新进程 context 的 ra 和 sp 与父进程不同。所以你不能在内核中通过 fork 和 exec 的简单组合实现 spawn。 
    - 在 spawn 中不应该有任何形式的内存拷贝。
    - spawn **不必** 像 fork 一样复制父进程的地址空间。

问答作业
--------------------

1. fork + exec 的一个比较大的问题是 fork 之后的内存页/文件等资源完全没有使用就废弃了，针对这一点，有什么改进策略？

2. [选做，不占分] 其实使用了题(1)的策略之后，fork + exec 所带来的无效资源的问题已经基本被解决了，但是今年来 fork 还是在被不断的批判，那么到底是什么正在"杀死"fork？回答合理即可。可以参考 `fork-hotos19 <https://www.microsoft.com/en-us/research/uploads/prod/2019/04/fork-hotos19.pdf>`_ 。

3. 请阅读代码并分析如下程序的输出，不考虑运行错误，不考虑行缓冲：
    
    .. code-block:: c 

        int main() {
            int val = 2;
            
            printf("%d", 0);
            int pid = fork();
            if (pid == 0) {
                val++;
                printf("%d", val);
            } else {
                val--;
                printf("%d", val);
                wait(NULL);
            }
            val++;
            printf("%d", val);
            return 0;
        }

    如果 fork() 之后一定主程序先运行结果如何？如果 fork() 之后一定 child 先运行结果如何？


4. 请分析如下程序运行后最终输出 ``A`` 的数量 (已知 ``||`` 的优先级比 ``&&`` 高)：

    .. code-block:: c 

        int main() {
            fork() && fork() && fork() || fork() && fork() || fork() && fork();
            printf("A");
            return 0;
        }

    [选做，不占分] 更进一步，如果给出一个 ``&&`` ``||`` 的序列，你如何设计一个程序来得到答案？

报告要求
--------------------

- [暂未支持] ``lab3.pdf`` CI 网站提交，注明姓名学号。 
- 注意目录要求，报告命名 ``lab3.md`` 或 ``lab3.pdf``，位于 ``reports`` 目录下。命名错误视作没有提交。不需要删除 ``lab1.md/pdf`` ``lab2.md/pdf``。后续实验同理。
- 完成 ch5 问答作业。
- [可选，不占分] 你对本次实验设计及难度的看法。