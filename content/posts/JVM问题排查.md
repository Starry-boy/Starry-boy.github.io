# JVM问题排查

JVM 工具

jvisualvm

jps

jstack

vmid

jstat

jinfo

jmap

JConsole

VisualVM

jmap -histo

## 一、CPU占用过高

1. 使用 top 查询 CPU 占用过高的进程

   example 查询占cpu最高的5个进程,并且每隔10秒刷新且只刷新一次.

   ```bash
   top -m 5 -d 10 -n 1 -s cpu
   ```

   | 参数 | 解释                                     | 实例                     |
   | ---- | ---------------------------------------- | ------------------------ |
   | -m   | max_procs最多显示多少个进程              | -m 1 显示1个进程         |
   | -n   | iterations 刷新次数                      | -n 10 只输出10次         |
   | -d   | delay 刷新的间隔时间，单位是秒 默认是5秒 | -d 10 每隔10秒刷新一次   |
   | -s   | 输出的数据按照那一列排序                 | -s cpu 标识按照CPU排序。 |
   | -t   | 显示线程信息，而不是进程。               |                          |
   | -h   | 显示帮助文档。                           |                          |

2. 查看 JVM 堆 内存使用信息

   ```bash
   jmap -heap <pid>
   ```

3. 查看 GC 信息

   ```bash
   jstat -gc <pid> 500 20
   ```

   ![image-20200818174629599](.image/image-20200818174629599.png)

   | 标题 | 描述                                                  |
   | ---- | ----------------------------------------------------- |
   | S0C  | 年轻代中第一个survivor（幸存区）的容量 (字节)         |
   | S1C  | 年轻代中第二个survivor（幸存区）的容量 (字节)         |
   | S0U  | 年轻代中第一个survivor（幸存区）目前已使用空间 (字节) |
   | S1U  | 年轻代中第二个survivor（幸存区）目前已使用空间 (字节) |
   | EC   | 年轻代中Eden（伊甸园）的容量 (字节)                   |
   | EU   | 年轻代中Eden（伊甸园）目前已使用空间 (字节)           |
   | OC   | Old代的容量 (字节)                                    |
   | OU   | Old代目前已使用空间 (字节)                            |
   | PC   | Perm(持久代)的容量 (字节)                             |
   | PU   | Perm(持久代)目前已使用空间 (字节)                     |
   | YGC  | 从应用程序启动到采样时年轻代中gc次数                  |
   | YGCT | 从应用程序启动到采样时年轻代中gc所用时间(s)           |
   | FGC  | 从应用程序启动到采样时old代(全gc)gc次数               |
   | FGCT | 从应用程序启动到采样时old代(全gc)gc所用时间(s)        |
   | GCT  | 从应用程序启动到采样时gc用的总时间(s)                 |

4. 查看 java 线程信息

   ```bash
   top -Hp <pid>
   ```

5. 将 PId 转 16进制

   ```bash
   printf %x <pid>
   ```

6. 查看堆栈信息

   ```bash
   jstack <pid>
   ```

7. 下载 dump 文件

   ```bash
   jmap -dump:fromat=b,file=heapDump <pid>
   ```

8. 堆外内存

   ![image-20210307165337888](.images/image-20210307165337888.png)

   tomcat修改配置 `-XX:NativeMemoryTracking=detail` 然后重启

   ```
   jcmd <pid> VM.native_memory detail scale=MB
   ```
   
   参考文章
   
   https://blog.csdn.net/hellozhxy/article/details/95203462
   
   
   https://bugs.java.com/bugdatabase/view_bug.do?bug_id=8180048
   
9. 统计线程数量

   ```bash
   jstack <pid> | grep 'tid' | wc -l
   ```

10. 查看内存分布

    ```bash
    pmap -x <pid> | sort -k 3 -n -r
    ```

11. 查看申请内存的线程

    ```bash
    
    ```
    
12. gperftools去定位问题

    ```
    
    ```

13. 用mat工具进行分析

    - 下载地址 https://www.eclipse.org/mat/downloads.php

    - mac mat 启动报错

      > 解决办法: 右键mat显示包内容，进入Contents->MacOS下面，会有一个MemoryAnalyzer的命令。
      >
      > 打开终端，进入此路径找到MemoryAnalyzer，运行
      >
      > ./MemoryAnalyzer -data dump文件所在文件夹路径

14. 查看linux 自动kill 掉的进程

    ```
    grep "Out of memory" /var/log/messages
    ```

15. 禁止 linux kill掉进程

    ```
    echo -17 > /proc/<pid>/oom_adj
    ```

    






