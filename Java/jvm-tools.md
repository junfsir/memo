### jps：虚拟机进程状况工具

*列出正在运行的虚拟机进程，并显示虚拟机执行主类（MainClass，main()函数所在的类）名称以及这些进程的本地虚拟机唯一ID（LVMID，Local Virtual Machine Identifier）；*

命令格式：

```shell
# jps [ options ] [ hostid ]
# jps可以通过RMI协议查询开启了RMI服务的远程虚拟机进程状态，参数hostid为RMI注册表中的主机名；
```

主要选项：

| 选项 |                         说明                         |
| :--: | :--------------------------------------------------: |
|  -q  |             只输出LVMID，省略主类的名称              |
|  -m  |    输出虚拟机进程启动时传递给主类main()函数的参数    |
|  -l  | 输出主类的全名，如果进程执行的是jar包，则输出jar路径 |
|  -v  |            输出虚拟机进程启动时的JVM参数             |

### jstat：虚拟机统计信息监视工具

*用于监视虚拟机各种运行状态信息的命令行工具；*

命令格式：

```shell
# jstat [ option vmid [interval[s|ms] [count]] ]
```

主要选项：

|       选项        |                             说明                             |
| :---------------: | :----------------------------------------------------------: |
|      -class       |      监视类加载、卸载数量、总空间以及类装载所耗费的时间      |
|        -gc        | 监视java堆状况，包括Eden区、2个Survivor区、老年代、永久代等的容量，已用空间，垃圾收集时间合计等信息 |
|    -gccapacity    | 监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间 |
|      -gcutil      | 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比 |
|     -gccause      | 与-gcuti功能一样，但是会额外输出导致上一次垃圾收集产生的原因 |
|      -gcnew       |                    监视新生代垃圾收集状况                    |
|  -gcnewcapacity   | 监视内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间 |
|      -gcold       |                    监视老年代垃圾收集状况                    |
|  -gcoldcapacity   | 监视内容与-gcold基本相同，输出主要关注使用到的最大、最小空间 |
|  -gcpermcapacity  |                 输出永久代使用的最大最小空间                 |
|     -compiler     |            输出即时编译器编译过的方法、耗时等信息            |
| -printcompilation |                   输出已经被即时编译的方法                   |

### jstack：Java堆栈跟踪工具

*生成虚拟机当前时刻的线程快照（一般称为threaddump或者javacore文件）；*

命令格式：

```shell
# jstack [ option ] vmid
```

主要选项：

| 选项 |                     说明                     |
| :--: | :------------------------------------------: |
|  -F  | 当正常输出的请求不被响应时，强制输出线程堆栈 |
|  -l  |        除堆栈外，显示关于锁的附加信息        |
|  -m  | 如果调用到本地方法的话，可以显示C/C++的堆栈  |

### jmap：Java内存映像工具

*用于生成堆转储快照（一般称为heapdump或dump文件）；*

命令格式：

```shell
# jmap [ option ] vmid
```

主要选项：

|      选项      |                             说明                             |
| :------------: | :----------------------------------------------------------: |
|     -dump      | 生成Java堆转储快照，格式为-dump:[live,]format=b,file=<filename>，其中live子参数说明是否只dump出存活对象 |
| -finalizerinfo |    显示在F-Queue中等待Finalizer线程执行finalize方法的对象    |
|     -heap      |  显示Java堆详细信息，如使用那种回收器，参数配置、分代状况等  |
|     -histo     |       显示堆中对象统计信息，包括类，实例数量，合计容量       |
|   -permstat    |          以ClassLoader为统计口径显示永久代内存状态           |
|       -F       | 当虚拟机进程对-dump选项没有响应时，可使用这个选项强制生成dump快照 |


