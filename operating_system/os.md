## OS

### 内核分类

微内核 - 弹性部署、5G、 IoT

宏内核 - PC、 phone

外核 - 科研、实验中、为应用定制操作系统 (多租户 request-based GC JVM)

### 用户态与内核态

cpu分不同的指令级别：linux内核跑在ring 0级， 用户程序跑在ring 3，对于系统的关键访问，需要经过kernel的同意，保证系统健壮性。

内核执行的操作 - > 200多个系统调用 sendfile read write pthread fork 

JVM -> 站在OS老大的角度，就是个普通程序

### 进程 线程 纤程 中断

面试高频：进程和线程有什么区别？

答案：进程就是一个程序运行起来的状态，线程是一个进程中的不同的执行路径。专业：进程是OS分配资源的基本单位，线程是执行调度的基本单位。分配资源最重要的是：独立的内存空间，线程调度执行（线程共享进程的内存空间，没有自己独立的内存空间）

纤程：用户态的线程，线程中的线程，切换和调度不需要经过OS

优势：1）占有资源很少 OS 的线程内存1M，Fiber只需要4K 

​		   2）切换比较简单

​           3）启动很多个10W+

目前（2020年3月22日）支持内置纤程的语言：Kotlin Scala Go Python(lib)... Java? （open jdk : loom）

### Java中对于纤程的支持：没有内置，盼望内置

使用Quaser库（不成熟），**测试下述代码并未发现纤程的速度比JVM线程快**。

```java
 <dependency>
            <groupId>co.paralleluniverse</groupId>
            <artifactId>quasar-core</artifactId>
            <version>0.8.0</version>
 </dependency>
```

创建普通的线程运行

```java
package fiber;

public class NormalThread {
    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis();
        Runnable r = new Runnable() {
            @Override
            public void run() {
                calc();
            }
        };

        int size = 10000;

        Thread[] threads = new Thread[size];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(r);
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].join();
        }

        long end = System.currentTimeMillis();
        System.out.println(end - start);
    }

    static void calc() {
        int result = 0;
        for (int m = 0; m < 10000; m++) {
            for (int i = 0; i < 200; i++) result += i;
        }
    }
}
```

使用纤程进行计算

```java
package fiber;


import co.paralleluniverse.fibers.Fiber;
import co.paralleluniverse.fibers.SuspendExecution;
import co.paralleluniverse.strands.SuspendableRunnable;

public class FiberTest {
    public static void main(String[] args) throws  Exception {
        long start = System.currentTimeMillis();

        int size = 10000;

        Fiber[] fibers = new Fiber[size];

        for (int i = 0; i < fibers.length; i++) {
            fibers[i] = new Fiber<>(new SuspendableRunnable() {
                public void run() throws SuspendExecution, InterruptedException {
                    calc();
                }
            });
        }

        for (int i = 0; i < fibers.length; i++) {
            fibers[i].start();
        }

        for (int i = 0; i < fibers.length; i++) {
            fibers[i].join();
        }

        long end = System.currentTimeMillis();
        System.out.println(end - start);


    }

    static void calc() {
        int result = 0;
        for (int m = 0; m < 10000; m++) {
            for (int i = 0; i < 200; i++) result += i;
        }
    }
}
```

作业：上述是使用10000个Fiber，1个JVM线程，想办法提高效率，如10000个Fiber，用10个JVM线程

```java
package fiber;


import co.paralleluniverse.fibers.Fiber;
import co.paralleluniverse.fibers.SuspendExecution;
import co.paralleluniverse.strands.SuspendableRunnable;

import java.util.concurrent.ExecutionException;

public class FiberTest2 {
    public static void main(String[] args) throws  Exception {
        long start = System.currentTimeMillis();
        Thread[] threads = new Thread[10];
        for (int i = 0; i < threads.length; i++) {
             threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        runFiber();
                    } catch (ExecutionException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].join();
        }

        long end = System.currentTimeMillis();
        System.out.println(end - start);


    }

    private static void runFiber() throws ExecutionException, InterruptedException {
        int size = 1000;
        Fiber[] fibers = new Fiber[size];

        for (int j = 0; j < fibers.length; j++) {
            fibers[j] = new Fiber<>(new SuspendableRunnable() {
                public void run() throws SuspendExecution, InterruptedException {
                    calc();
                }
            });
        }

        for (int j = 0; j < fibers.length; j++) {
            fibers[j].start();
        }

        for (int j = 0; j < fibers.length; j++) {
            fibers[j].join();
        }
    }

    static void calc() {
        int result = 0;
        for (int m = 0; m < 10000; m++) {
            for (int i = 0; i < 200; i++) result += i;
        }
    }
}
```

###  纤程的应用场景

纤程 VS 线程池：很短的计算任务，不需要和内核打交道，并发量高。

### Unix/Linux进程

在Unix或Linux中，正常情况下，子进程是通过父进程fork出来的，子进程可以再创建新的进程。

子进程的结束和父进程的运行是一个异步过程，即父进程永远无法预测子进程什么时候会结束。

当一个进程完成它的工作并结束后，它的父进程需要调用wait或waitpid系统调用才会取得子进程的终止状态。

unix可以保证父进程想知道子进程结束时的状态信息，机制：在每个进程退出的时候，内核释放该进程的所有资源，包括打开的文件、占用的内存等。但是仍然为其保留一定的信息（包括进程号、退出状态、运行时间等），直到父进程通过wait/waitpid来取时才会释放。

### 僵尸进程

**一个进程使用fork创建子进程，如果子进程退出，而父进程没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称为僵死进程。**

僵死进程可能会导致的问题：

如果父进程不调用wait或waitpid，那么子进程退出时保留的信息就不会释放（进程号等），但是系统的进程号是有限的，如果产生大量的僵死进程，那么就会导致系统没有可用的进程号，无法产生新的进程。

测试程序如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <assert.h>
#include <sys/types.h>
int main() {
        pid_t pid = fork();
  //在执行fork函数之前，只有当前进程在执行这段代码，但是这条语句被执行之后，后面的语句就变成两个进程在执行了，fork函数的一个独特之处是被调用一次，但是却会返回两次，有三种返回值
  		//在父进程中，fork函数返回子进程的pid
  		//在子进程中，返回0
  		//如果出现错误，fork就会返回一个负值
        if (0 == pid) {
                printf("child id is %d\n", getpid());
                printf("parent id is %d\n", getppid());
        } else {
                while(1) {}
        }
}
```

### 孤儿进程

孤儿进程：**一个父进程退出，而它的一个或多个子进程还在运行，这些子进程就称为孤儿进程，孤儿进程会被init进程（进程号为1）收养，并由init进程对它们完成状态的收集。**

**孤儿进程是没有父进程的进程，孤儿进程这个重任就落到了init进程身上**，每当出现一个孤儿进程的时候，内核就把孤 儿进程的父进程设置为init，而init进程会循环地wait()它的已经退出的子进程。**因此孤儿进程并不会有什么危害。**

测试程序如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <assert.h>
#include <sys/types.h>

int main() {
        pid_t pid = fork();

        if (0 == pid) {
                printf("child ppid is %d\n", getppid());
                sleep(10);
                printf("parent ppid is %d\n", getppid());
        } else {
                printf("parent id is %d\n", getpid());
                sleep(5);
                exit(0);
        }
}
```

### 进程调度

Linux2.6采用CFS调度策略：Completely Fair Scheduler

按优先级分配时间片的比例，记录每个进程的执行时间，如果有一个进程执行时间不到他应该分配的比例，优先执行

默认调度策略：

实时 （急诊） 优先级分高低 - FIFO (First In First Out)，优先级一样 - RR（Round Robin） 普通： CFS

### 中断

中断：硬件跟操作系统内核打交道的一种机制，其中软中断（80中断） ==  系统调用

系统调用：int 0x80 或者 sysenter原语，通过ax寄存器填入调用号，参数通过bx cx dx si di传入内核，返回值通过ax返回 

java读网络 – jvm read() – c库read() - > 内核空间 -> system_call() （系统调用处理程序）-> sys_read()

### 从汇编角度理解软中断

搭建汇编环境

1.安装nasm，yum install nasm

2.测试程序

```asm
;hello.asm
;write(int fd, const void *buffer, size_t nbytes)
;fd 文件描述符 file descriptor - linux下一切皆文件

section data
    msg db "Hello", 0xA
    len equ $ - msg

section .text
global _start
_start:

    mov edx, len
    mov ecx, msg
    mov ebx, 1 ;文件描述符1 std_out
    mov eax, 4 ;write函数系统调用号 4
    int 0x80

    mov ebx, 0
    mov eax, 1 ;exit函数系统调用号
    int 0x80
```

3.编译：nasm -f elf  hello.asm -o hello.o

4.链接：ld -m elf_i386 -o hello hello.o

一个程序的执行过程，要么处于用户态，要么处于内核态