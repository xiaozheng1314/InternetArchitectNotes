读书原则：不求甚解，观其大略。

## 书籍推荐

▪《编码：隐匿在计算机软硬件背后的语言》

▪《深入理解计算机系统》

▪ 语言：C JAVA  K&R《C程序设计语言》《C Primer Plus》

▪ 数据结构与算法： -- 毕生的学习 leetCode

​          –《Java数据结构与算法》《算法》

​          –《算法导论》《计算机程序设计艺术》//难

▪ 操作系统：Linux内核源码解析  Linux内核设计与实现 30天自制操作系统

▪ 网络：机工《TCP/IP详解》卷一 翻译一般

▪ 编译原理：机工 龙书 《编译原理》 《编程语言实现模式》马语

▪ 数据库：SQLite源码 Derby - JDK自带数据库

## 硬件基础知识

### CPU的制作过程

Intel cpu的制作过程

[https://haokan.baidu.com/v?vid=11928468945249380709&pd=bjh&fr=bjhauthor&type=](https://haokan.baidu.com/v?vid=11928468945249380709&pd=bjh&fr=bjhauthor&type=video)[video](https://haokan.baidu.com/v?vid=11928468945249380709&pd=bjh&fr=bjhauthor&type=video)

CPU是如何制作的（文字描述）

[https](https://www.sohu.com/a/255397866_468626)[://www.sohu.com/a/255397866_468626](https://www.sohu.com/a/255397866_468626)

### CPU的原理

计算机需要解决的最根本问题：如何代表数字

晶体管是如何工作的：

[https://haokan.baidu.com/v?vid=16026741635006191272&pd=bjh&fr=bjhauthor&type=](https://haokan.baidu.com/v?vid=16026741635006191272&pd=bjh&fr=bjhauthor&type=video)[video](https://haokan.baidu.com/v?vid=16026741635006191272&pd=bjh&fr=bjhauthor&type=video)

晶体管的工作原理：

https://www.bilibili.com/video/av47388949?p=2

### 汇编语言（机器语言）的执行过程

汇编语言的本质：机器语言的助记符，其实也就是机器语言

计算机通电 -> CPU读取内存中程序（电信号输入）-> 时钟发生器不断震荡通断电 -> 推动CPU内部一步一步执行

（执行多少步取决于指令需要的时钟周期）-> 计算完成->写回（电信号）-> 写给显卡输出（sout，或者图形）

### 量子计算机

量子比特，同时表示1 0

### CPU的基本组成

**PC（Program Counter）**： 程序计数器，记录当前的指令地址

**Registers** ：暂时存储CPU计算需要用到的数据

**ALU（Arithmetic & Logic Unit）运算单元**

**CU（Control Unit）控制单元**

**MMU（Memory Management Unit）内存管理单元**

cache

### 缓存

MESI：Intel的缓存一致性协议

https://www.cnblogs.com/z00377750/p/9180644.html

缓存行（cache line）：目前大小多为64字节

​		缓存行越大，局部性空间效率越高，但读取时间慢

​		缓存行越小，局部性空间效率越低，但读取时间快

缓存行对齐：对于某些特别敏感的数字会存在线程高竞争的访问，为了保证不发生伪共享，就可以使用缓存行对齐的编程方式。测试代码如下：

```java
package cacheline;

public class NoCacheLinePadding {
    public static volatile long[] arr = new long[2];

    public static void main(String[] args) throws Exception {
        Thread t1 = new Thread(() -> {
            for (long i = 0; i < 10000_0000L; i++) {
                arr[0] = i;
            }
        });

        Thread t2 = new Thread(() -> {
            for (long i = 0; i < 10000_0000L; i++) {
                arr[1] = i;
            }
        });

        final long start = System.nanoTime();
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println((System.nanoTime() - start) / 100_0000);
    }
}
```

```java
package cacheline;

public class CacheLinePadding {
    public static volatile long[] arr = new long[16];

    public static void main(String[] args) throws Exception {
        Thread t1 = new Thread(()->{
            for (long i = 0; i < 10000_0000L; i++) {
                arr[0] = i;
            }
        });

        Thread t2 = new Thread(()->{
            for (long i = 0; i < 10000_0000L; i++) {
                arr[8] = i;
            }
        });

        final long start = System.nanoTime();
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println((System.nanoTime() - start)/100_0000);
    }
}
```

在JDK7中，很多都是采用long padding提高效率；在JDK8中，可以使用@Contended注解加上JVM的-XX：-RestrictContended参数。



